---
layout: post
title: Inline Query
---


![_config.yml]

### Add gem

```
gem "schema_plus_views"
```

### Create SQL View superclass

```
class SqlView < ActiveRecord::Base
  self.abstract_class = true
  self.primary_key = :id
  after_initialize :readonly!
end
```

### Create SQL View model
```
class Leaderboard < SqlView
  belongs_to :competition
  belongs_to :user
end
```

Paste in the SQL commented out as a reference.

### Migration

```
class CreateLeaderboardSqlView < ActiveRecord::Migration
  def change
    #drop_view :leaderboards
    create_view :leaderboards,
    "
    SELECT s.id, s.competition_id, s.user_id, NULL AS team_id, s.score, cnt.entries
    FROM submissions s,
        (SELECT competition_id,
                user_id,
                team_id,
                count(*) AS entries
          FROM submissions
        GROUP BY competition_id, user_id, team_id) cnt
    WHERE s.evaluated = TRUE
    AND s.user_id = cnt.user_id
    AND s.competition_id = cnt.competition_id
    AND s.score = (SELECT max(m.score)
                     FROM submissions m
                   WHERE m.competition_id = s.competition_id
                     AND m.user_id = s.user_id
                     AND m.evaluated = TRUE)
    "
  end
end
```

Get existing SQL view text
```
select pg_get_viewdef('viewname', true)
```

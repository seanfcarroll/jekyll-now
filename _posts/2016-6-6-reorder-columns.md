---
layout: post
title: Update physical column order
---


Reorder the columns on an existing table

```
class Reorder < ActiveRecord::Migration
  def change
    execute "CREATE TABLE events_tmp AS SELECT id, challenge_id, event, seq, event_time, created_at, updated_at FROM events;"
    execute "drop table events;"
    execute "create table events as select * from events_tmp;"
    execute "drop table events_tmp;"
  end
end
```

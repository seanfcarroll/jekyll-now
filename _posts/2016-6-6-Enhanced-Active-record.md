---
layout: post
title: Enhanced active record queries
---

```
@introductions = choose_introductions(current_user,@user.id)

  def choose_introductions(current_user,user_id)
      sql = %Q[
          select id, user_id, language_cd, level_cd, native, tagline,
                      introduction, teaching, evaluated,
                      created_at, updated_at
          from (
            select 1, l.*
              from user_languages l
              where l.user_id = #{user_id}
              and l.language_cd in ('#{current_user.fluent_language_cd}',
                                    '#{current_user.target_language_cd}')
              or l.language_cd = 'en'
            union
            select 2, l.*
              from user_languages l
              where l.user_id = #{user_id}
              and l.language_cd not in ('fr','en')
              and l.native = true
            union
            select 3, l.*
              from user_languages l
              where l.user_id = #{user_id}
              and l.language_cd not in ('fr','en')
              and l.native = false
            ) a
            where a.introduction != ''
            order by 1
            limit 2
          ]
      return UserLanguage.find_by_sql(sql)
    end
```

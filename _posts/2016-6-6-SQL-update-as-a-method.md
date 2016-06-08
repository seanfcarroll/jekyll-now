---
layout: post
title: SQL Update as a method
---

A complex update statement is abstracted as a Ruby method, and is passed paramaters. Return values are possible, including 





.

```
   def update_cache_hits(blast_cache_ids)
      sql = %Q[
         update blast_caches bc
         set significant_hits = wrk2.significant,
         insignificant_hits = wrk2.insignificant
         from
             (select blast_cache_id, 
                     sum(significant) as significant, 
                     sum(insignificant) as insignificant
             from 
                 (select blast_cache_id, count(*) as significant, 0 as insignificant
                 from  work_blast_selections
                 where blast_cache_id in (#{blast_cache_ids})
                 and hit_type = 'significant'
                 group by blast_cache_id
                 union
                 select blast_cache_id, 0 as significant, count(*) as insignificant
                 from work_blast_selections
                 where blast_cache_id in (#{blast_cache_ids})
                 and hit_type = 'insignificant'
                 group by blast_cache_id) wrk
             group by blast_cache_id ) wrk2
          where wrk2.blast_cache_id = bc.id
      ]
      @logger.log(sql)
      result = ActiveRecord::Base.connection.execute(sql)
   end
   ```

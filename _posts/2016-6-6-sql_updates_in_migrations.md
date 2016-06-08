---
layout: post
title: SQL Updates in migrations
---


![_config.yml]



```
class CopyFirstNameToName < ActiveRecord::Migration
  def up
    execute "update users set name = first_name"
  end

  def down
    raise ActiveRecord::IrreversibleMigration
  end
end
```


Connect directly to PG and MySQL databases using a Ruby script

```
require 'mysql2'
require 'pg'


conn = Mysql2::Client.new(:host => "localhost", :username => "root", :password => 'thepasswoed', :database => 'hg19')
pg_conn = PG.connect(dbname: 'assay_development')

# predelete
pg_conn.exec('delete from genes')
pg_conn.exec('delete from exons')

results = conn.query('select * from refGene')
results.each do |row|
  genes_id = pg_conn.exec('insert into genes (name, name2, chrom, chrom_start, chrom_end, exon_count, ' + 
                          'exon_start, exon_end, created_at, updated_at) values ' + 
                          '($1, $2, $3, $4, $5, $6, $7, $8, current_date, current_date) returning id', 
                          [row['name'],
                          row['name2'],
                          row['chrom'],
                          row['cdsStart'],
                          row['cdsEnd'],
                          row['exonCount'],
                          row['txStart'],
                          row['txEnd']])
  puts "id: #{genes_id[0]['id']}"
                          
  start_array = row['exonStarts'].split(',').collect{|s| s.to_i }
  end_array = row['exonEnds'].split(',').collect{|s| s.to_i }
  start_array.each_with_index do |elem, index|
    exons_res = pg_conn.exec('insert into exons (genes_id, chrom, chrom_start, chrom_end, created_at, updated_at) ' +
                             'values ($1, $2, $3, $4, current_date, current_date) returning id',
                             [genes_id[0]['id'],
                              row['chrom'],
                              start_array[index],
                              end_array[index]])                         
    puts row['name'], start_array[index], end_array[index]
  end
  puts
end
```

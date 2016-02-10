# Select extreme rows within groups

I want to select rows from a collection where one column
is the maximum/minimum within a group.

In Pandas:

```python

In [7]: df = pd.DataFrame(dict(a=[1, 1, 2, 2], b=[10, 20, 30, 20]))
In [8]: df.groupby('a').apply(lambda grp: grp[grp.b == grp.b.max()])
Out[8]:
     a   b
a
1 1  1  20
2 2  2  30
```

To do the same in PostgreSQL, you can use `DISTINCT ON` with `ORDER BY` to
put the relevant row at the top of each group boundary, and
to select the first row in each group.


```
SELECT DISTINCT ON (a) a, b, c from table ORDER by a, b
```

(note that the sql example selects an arbitrary single row in the event of ties, the pandas snippet would return all rows)

`DISTINCT ON` guarantees that the first match will be the one returned.

This is also exposed in Django

```python
Table.objects.order_by('a', 'b').distinct('a').values_list('a', 'b', 'c')
```

One potential downside to this that you end up sorting
the entire groupset, instead of sorting within groups:


```
Unique  (cost=11838.80..11843.31 rows=601 width=23) (actual time=1104.219..1122.251 rows=1301 loops=1)
  ->  Sort  (cost=11838.80..11840.30 rows=601 width=23) (actual time=1104.217..1110.002 rows=79887 loops=1)
        Sort Key: wetlab_batch.name, sequencing_lanesample_seqalleles.seqallele_id, sequencing_flowcell.date
        Sort Method: quicksort  Memory: 9314kB
        ->  Nested Loop  (cost=0.02..11811.06 rows=601 width=23) (actual time=0.446..1014.528 rows=79887 loops=1)
              ->  Nested Loop  (cost=0.00..1415.78 rows=103 width=23) (actual time=0.392..108.090 rows=12891 loops=1)
                    ->  Nested Loop  (cost=0.00..928.86 rows=103 width=12) (actual time=0.383..70.111 rows=12891 loops=1)
                          ->  Nested Loop  (cost=0.00..73.06 rows=118 width=12) (actual time=0.363..10.049 rows=13632 loops=1)
                                ->  Seq Scan on sequencing_flowcell  (cost=0.00..56.90 rows=1 width=8) (actual time=0.338..0.557 rows=128 loops=1)
                                      Filter: (date > '2016-01-10'::date)
                                      Rows Removed by Filter: 2364
                                ->  Index Scan using sequencing_lanesample_flowcell_id on sequencing_lanesample  (cost=0.00..14.91 rows=125 width=12) (actual time=0.008..0.041 rows=106 loops=128)
                                      Index Cond: (flowcell_id = sequencing_flowcell.id)
                          ->  Index Scan using wetlab_well_pkey on wetlab_well  (cost=0.00..7.24 rows=1 width=8) (actual time=0.003..0.004 rows=1 loops=13632)
                                Index Cond: (id = sequencing_lanesample.well_id)
                                Filter: for_production
                                Rows Removed by Filter: 0
                    ->  Index Scan using wetlab_batch_pkey on wetlab_batch  (cost=0.00..4.72 rows=1 width=19) (actual time=0.002..0.002 rows=1 loops=12891)
                          Index Cond: (id = wetlab_well.batch_id)
              ->  Index Only Scan using sequencing_lanesample_seqall_lanesample_id_b9cbf58ce9ed9e2_uniq on sequencing_lanesample_seqalleles  (cost=0.02..100.87 rows=6 width=8) (actual time=0.010..0.066 rows=6 loops=12891)
                    Index Cond: ((lanesample_id = sequencing_lanesample.id) AND (seqallele_id = ANY ('{221729,802,740,806,774,796,746,476919,836621,458832,35601,786,30035,134377,20440,794,20444,20442,766,836607}'::integer[])))
                    Heap Fetches: 79887
```

## Sources

 * [SO Question](http://stackoverflow.com/a/28090544)
 * [PostgreSQL docs](http://www.postgresql.org/docs/9.2/static/sql-select.html)
 * [Slack thread](https://counsyl.slack.com/archives/databases/p1455055606000002)

h/t @cmason, @ihaque

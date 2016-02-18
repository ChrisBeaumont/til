## Minimalist database access with Records

[Records](https://github.com/kennethreitz/records) is a great library for running simple sql queries.


```python
In [22]: db = records.Database('sqlite:///clinvar.db')

In [23]: db.query('select * from allele limit 2').all()
Out[23]:
[<Record {"id": "ff3f1090e586617d694b8cddbd428136328cb7ae", "genome": "GRCh38", "chrom": "chr3", "start": 150928107, "end": 150928107, "ref": "A", "alt": "C", "normalization_needed": 0}>,
 <Record {"id": "e343886b2d0f060b28418101f9774cfc15e2bc74", "genome": "GRCh37", "chrom": "chr3", "start": 150645894, "end": 150645894, "ref": "A", "alt": "C", "normalization_needed": 0}>]
 ```

Fields in `Record` objects are addressible via name or position index.


It doesn't seem to provide database schema introspection besides `show_tables`

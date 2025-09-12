
# 1617957.pts-53.tardis
```bash
nohup RepeatModeler -database ragtag_db -pa 20 >& ragtag_run.out &
nohup RepeatModeler -database canu_db -pa 20 >& canu_run.out &

```

```bash
(base) [aki@tardis ~]$ nohup RepeatModeler -database ragtag_db -pa 20 >& ragtag_run.out &
[1] 1618113
(base) [aki@tardis ~]$ nohup RepeatModeler -database canu_db -pa 20 >& canu_run.out &
[2] 1618118
(base) [aki@tardis ~]$ tail -f ragtag_run.out
  - Sequences = 11
  - Bases = 34861082
Using output directory = /home/aki/RM_1618115.FriSep121014062025


RepeatModeler Round # 1
========================
Searching for Repeats
 -- Sampling from the database...
   - Gathering up to 40000000 bp

^C
(base) [aki@tardis ~]$ tail -f ragtag_run.out
  - Sequences = 11
  - Bases = 34861082
Using output directory = /home/aki/RM_1618115.FriSep121014062025


RepeatModeler Round # 1
========================
Searching for Repeats
 -- Sampling from the database...
   - Gathering up to 40000000 bp
^C
(base) [aki@tardis ~]$ tail -f canu_run.out
  - Sequences = 29
  - Bases = 35997303
Using output directory = /home/aki/RM_1618120.FriSep121014122025


RepeatModeler Round # 1
========================
Searching for Repeats
 -- Sampling from the database...
   - Gathering up to 40000000 bp
^C
(base) [aki@tardis ~]$ 
```






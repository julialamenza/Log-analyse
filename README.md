Logs Analysis
=================================

P3: Logs Analysis

This project's goal was to build an internal command line reporting tool that uses information queried from a newspaper site's database to determine what kind of articles the newspaper site's readers prefer. The tool determines the most popular articles, as well as authors, and days where the site experienced a high level of user request errors based on user activity and web server logs.


Requirements
------------

+ [Python 3](https://www.python.org/downloads/) is installed.
+ This can be verified by running the following command in the terminal:
```bash
$ python -V
```
+ [Virtual Box](https://www.virtualbox.org/wiki/Downloads) is installed.
+ [Vagrant](https://www.vagrantup.com/downloads.html) is installed.
+ [VM configuration files](https://github.com/udacity/fullstack-nanodegree-vm) are setup.
+ [Git](https://git-scm.com/downloads) is installed.
  (Optional, if you wish to clone the project repository.)


Usage
-----

```bash
$ git clone https://github.com/davidsimowitz/fullstack-nanodegree-project-3.git
```
  + Above command is optional.
  + Alternatively you may download the files into the directory.
```bash
$ cd fullstack-nanodegree-project-3
```
  + Verify the following files are present before continuing:
    * summary_reporting_tool.py
    * newsdata.zip
    * fsnd-virtual-machine.zip
    * SUMMARY_REPORT_2017-07-14_01:09:19_UTC.log  (generated report example)

* Install Virtual Box, then Vagrant. (If not previously installed)
* Unzip the fsnd-virtual-machine.zip file (This contains the VM files).
* Unzip the newsdata.zip file (contains the PostgreSQL database for this project)

* Copy summary_reporting_tool.py and newsdata.sql to the vagrant directory in the VM.
```bash
$ cp summary_reporting_tool.py ./FSND-Virtual-Machine/vagrant/summary_reporting_tool.py
$ cp newsdata.sql ./FSND-Virtual-Machine/vagrant/newsdata.sql
```
* Enter the VM directory.
```bash
$ cd FSND-Virtual-Machine/vagrant/
```
* Startup the VM. (This may take a while)
```bash
$ vagrant up
```
* Log in to the VM.
```bash
$ vagrant ssh
```
* Enter the shared vagrant directory and load the data from newsdata.sql.
```bash
$ cd /vagrant
$ psql -d news -f newsdata.sql
```
* Run the summary_reporting_tool.py report generator from the command-line. The results will automatically be output to the terminal once the command is run. A log file will also be generated in the current directory.
```bash
$ ./summary_reporting_tool.py
```


###How to create Views
sThis project uses views that was created in BD and are reference by  `relatorio.py`. To this program works  correctly, this views should be created in the `news` database.
1) to create the view `artigos_mais_populares`:
```sql
create view artigos_mais_populares as
select articles.title, count(log.path) as views
from log, articles
where log.status = '200 OK' and log.path like concat('%', articles.slug)
group by log.path, articles.title
order by views desc limit 3;
```

2) to create the  view `autores_mais_populares`:
```sql
create view autores_mais_populares as
select authors.name, count(articles.author) as views
from log, articles, authors
where log.status = '200 OK' and log.path like concat('%', articles.slug) and articles.author=authors.id
group by articles.author, authors.name
order by views desc;
```

3) to create the  view `dias_com_mais_de_1_por_cento_de_erros`:
```sql
create view dias_com_mais_de_1_por_cento_de_erros as

select * from

(select to_char(ers.data, 'Mon DD, YYYY') as data, ers.erros::decimal / oks.validos * 100 as porcentagem from

(select date(log.time) as data, count(date(log.time)) as erros
from log
where log.status != '200 OK'
group by date(log.time)
order by erros desc ) as ers,

(select date(log.time) as data, count(date(log.time)) as validos
from log where log.status = '200 OK'
group by date(log.time)
order by validos desc ) as oks

where ers.data = oks.data

order by porcentagem desc) as consulta

where porcentagem > 1;
```

You can check this queries at repo as well.
### How to RUN
At the VM terminal run  `python relatorio.py`.wait and the file `relatorio.txt` will be updated.

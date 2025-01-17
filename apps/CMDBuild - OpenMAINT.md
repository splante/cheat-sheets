[CMDBuild](https://www.cmdbuild.org/en) is an open source environment for configuring customized applications for the Asset Management. OpenMAINT is a config implementation of CMDBuild for Property and Facility Management and Maintenance Processes.

```bash
[splante@openmaint openmaint]$ pwd
/home/splante/openmaint
[splante@openmaint openmaint]$ webapps/cmdbuild/cmdbuild.sh dbconfig recreate empty

[splante@openmaint openmaint]$ cat conf/cmdbuild/database.conf
#Fri Oct 07 15:53:40 EDT 2022
db.admin.password=************************
db.password=*********************
db.username=openmaint
db.admin.username=postgres
db.url=jdbc:postgresql://localhost:5432/openmaint
ext = plpgsql,postgis
vert.name = openMAINT
vert.version = 2.2-d

```

# Manuals
[Tech Manual - Local](obsidian://open?vault=cheat-sheets&file=apps%2Fcmdbuild%2Ftechnical-manual-in-english.pdf)  - [Latest/Remote](https://www.cmdbuild.org/file/manuali/technical-manual-in-english)

# External Software
## Required
[PostgreSQL](obsidian://open?vault=cheat-sheets&file=databases%2Fpostgres) with PostGIS 

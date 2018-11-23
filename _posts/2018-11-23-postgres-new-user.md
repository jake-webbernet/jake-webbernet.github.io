Had to create a user for a migration at work. 

This user needed to be able to add and view rows on the database needed for migration work.

I ran the below steps to get it setup

* Connected to Postgres via the root user
* Created a user that could be use for the migration

```sql
CREATE USER migrationuser WITH PASSWORD 'some-pasword';
```

* Exited the root user, and connected to user that owns the database that I want to grant access too (the following did not work from the root user)

* Ran the below

```sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO migrationuser;
GRANT INSERT ON ALL TABLES IN SCHEMA public TO migrationuser;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO migrationuser;
GRANT UPDATE ON ALL SEQUENCES IN SCHEMA public TO migrationuser;
```

From here the user was able to login and run INSERT and SELECT commands with no problems

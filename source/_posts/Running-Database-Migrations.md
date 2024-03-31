---
title: Running Database Migrations
tags:
  - architecture
date: 2024-03-31 11:10:44
---


Let's start with a question: how do you run database migrations? Depending on the technology you are using, you may choose something like [Flyway](https://documentation.red-gate.com/home), [Alembic](https://alembic.sqlalchemy.org/en/latest/), or some other tool that fits well with your process. My preference is to write and run the migrations as SQL files. I recently released a small library, [yoyo-cloud](https://pypi.org/project/yoyo-cloud/), that allows storing the migrations as files in S3 and then applying them to the database. In this post we will look at the rationale behind the library, and the kind of workflow it enables.  

## Before We Begin

We'll start with an example that shows how to run migrations on a Postgres instance. There are a couple of SQL files that I have stored in an S3 bucket — one to create a table, and another to insert a row after the table is created. The bucket is public so you should be able to run the snippet below.

{% code lang:python %}
from yoyo import get_backend
from yoyo_cloud import read_s3_migrations

if __name__ == "__main__":
    migrations = read_s3_migrations(paths=["s3://yoyo-migrations/"])
    backend = get_backend(f"postgresql://postgres:my-secret-pw@localhost:5432/postgres")

    with backend.lock():
        # -- Apply any outstanding migrations
        backend.apply_migrations(backend.to_apply(migrations))
{% endcode %}

As you can see from the imports, we use a combination of the original library, `yoyo`, which provides a helper function to connect to the database, and the new library, `yoyo_cloud`, which provides a helper function to read the migrations that are stored in S3. 

The `read_s3_migrations` function reads the files that are stored in S3. It takes as input a list of S3 paths that point to directories where the files are stored. This function is similar in interface to the `read_migrations` function in `yoyo` which reads migrations from a directory on the file system except that it reads from S3; the value returned is the same — a list of migrations.

Finally, we apply the migrations to the Postgres instance. The purpose of this example was to demonstrate the simplicity with which migrations stored in S3 can be applied to a database using `yoyo_cloud`. We will now look at the rationale behind the library in the next section.

## Rationale 

The rationale behind the library, as mentioned at the start of this post, is to store SQL migrations as files and apply them one-by-one. Additionally, it allows migrating multiple similar databases easily. In the [previous post on scaling Python microservices](2024/03/25/Scaling-Python-Microservices/) we'd seen how a Flask microservice can dynamically connect to multiple databases. If we want to apply a migration, say adding a new column to a table, we'd want it to be applied to all the databases that the service connects to. Using `yoyo_cloud` we can read the migration from S3, and apply to every table. Since the migrations are idempotent, we can safely reapply the previous migrations. 

## Workflow

Let's assume we'd like to create an automation which applies database migrations. We'll assume we have two environments — stage, and prod. Whenever a migration is to be released to production, it is first tested in the dev environment, and then committed to version control to be applied to the staging environment. These could be stored as a part of the same repository or in a separate repository that contains only migrations. Let's assume it is the latter. We could have a directory structure as follows.  

{% code %}
migrations
  - order_service
    - prod
      - 001-create-table.sql
    - stage
      - 001-create-table.sql
      - 002-add-column.sql
{% endcode %}

The automation could then apply these migrations to the relevant environment of the service. An additional benefit of this approach is that it makes setting up new environments easier because the migrations can applied to a new database. Additionally, committing migrations to version control, as opposed to running them in an adhoc manner, allows keeping track of when the migration was introduced, why, and by whom.

## Conclusion  

That's it. That's how we can apply migrations stored in S3 using `yoyo_cloud`. If you're using it, or considering using it, please leave a comment.
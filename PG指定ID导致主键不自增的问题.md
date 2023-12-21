
https://stackoverflow.com/questions/37970743/postgresql-unique-violation-7-error-duplicate-key-value-violates-unique-const


Postgres handles auto incrementing a little differently than MySQL does. In Postgres, when you create the `serial` field, you are also creating a sequence field that is keeping track of the id to use. This sequence field is going to start out with a value of 1.

When you insert a new record into the table, if you don't specify the `id` field, it will use the value of the sequence, and then increment the sequence. However, if you do specify the `id` field, then the sequence is not used, and it is not updated, either.

I'm assuming that when you moved over to Postgres, you seeded or imported some existing users, along with their existing ids. When you created these user records with their ids, the sequence was not used, and therefore it was never updated.

So, if, for example, you imported 10 users, you have users with ids 1-10, but your sequence is still at 1. When you attempt to create a new user without specifying the id, it pulls the value from the sequence (1), and you get a unique violation because you already have a user with id 1.

To resolve the issue, you need to set your `users_id_seq` sequence value to the MAX(id) of your existing users. You can read [this question/answer](https://stackoverflow.com/questions/244243/how-to-reset-postgres-primary-key-sequence-when-it-falls-out-of-sync) for more information on resetting the sequence, but you can also try something like (untested):

```sql
SELECT setval(pg_get_serial_sequence('users', 'id'), coalesce(max(id)+1, 1), false) FROM users;
```

---

FYI, this is not an issue in MySQL because MySQL automatically updates the auto increment sequence to the largest column value when a value is manually inserted into the auto incrementing field.
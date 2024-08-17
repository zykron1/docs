Database Guide
==============
A complete guide to how database modeling works in Mercury.

Creating a Model
----------------

To create a model in Mercury, you can use the CLI:

.. code-block:: bash

	python3 -m libmercury create model user

This command generates a model called `user`. You can find the generated code in `src/cargo/user.py`:

.. code-block:: python3

	from libmercury.db import Column, Integer, Base

	class user(Base):
		__tablename__ = "user"
		id = Column(Integer, primary_key=True)

Adding Columns to Your Model
----------------------------

Let's extend the `user` model by adding `username` and `password` columns:

.. code-block:: python3

	from libmercury.db import Column, Integer, String, Base

	class user(Base):
		__tablename__ = "user"
		id = Column(Integer, primary_key=True)
		username = Column(String(20))
		password = Column(String(50))

.. warning::

	Never store passwords in plaintext. Use hashing instead. Mercury has built-in tools for this.

Creating a Migration
--------------------

After creating the model, you need to sync the database schema with the current state of your models. To do this, create a migration:

.. code-block:: bash

	python3 -m libmercury create migration first_migration

This command generates a migration file in `src/cargo/migrations/1.py`. Let's take a look at its contents:

.. code-block:: python3

	from libmercury.db import MigrationWrapper 

	_version = '1'
	_prev_version = None
	
	def upgrade(url):
		wrapper = MigrationWrapper(url)
	
	def downgrade(url):
		wrapper = MigrationWrapper(url)

The `upgrade` function is where you define the changes to apply to the database schema. The `downgrade` function defines how to revert those changes.

Using the MigrationWrapper
--------------------------

Let's start by creating the `user` table:

.. code-block:: python3

	from libmercury.db import MigrationWrapper, Column, Integer, String 

	_version = '1'
	_prev_version = None
	
	def upgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.create_table("user", [
			Column("id", Integer, primary_key=True),
			Column("username", String(20)),
			Column("password", String(50))
		])
	
	def downgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.delete_table("user")

Running the Migration
---------------------

To apply the migration, use the CLI:

.. code-block:: bash

	python3 -m libmercury migrate

If successful, you'll see output similar to:

.. code-block:: bash

	[Migrator] Running migration src/cargo/migrations/1.py
	[Migrator] Table 'user' created successfully.
	[Migrator] 'src/cargo/migrations/1.py' passed with no errors

Making Schema Changes with Migrations
-------------------------------------

Now that you understand the basics, let's explore more advanced migration operations. The `MigrationWrapper` provides several functions to modify the database schema.

### Adding a Column

Suppose you want to add a new `email` column to the `user` table:

.. code-block:: python3

	def upgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.add_column("user", Column("email", String(50)))

	def downgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.drop_column("user", "email")

### Modifying a Column

To modify an existing column, such as changing the length of the `username` column from 20 to 50:

.. code-block:: python3

	def upgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.modify_column("user", "username", String(50))

	def downgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.modify_column("user", "username", String(20))

### Dropping a Column

If you need to remove a column, like the `password` column:

.. code-block:: python3

	def upgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.drop_column("user", "password")

	def downgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.add_column("user", Column("password", String(50)))

Deleting a table
----------------

To remove an entire table, use the `delete_table` function:

.. code-block:: python3

	def upgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.delete_table("user")

	def downgrade(url):
		wrapper.create_table("user", [
			Column("id", Integer, primary_key=True),
			Column("username", String(20)),
			Column("password", String(50))
		])

Conclusion
----------
With these functions, you can manage your database schema more effectively, making it easy to adapt as your application
evolves. 

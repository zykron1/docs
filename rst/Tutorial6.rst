Tutorial 6
==========
This tutorial will be based of the `previous tutorial <tutorial5.html>`_. Please complete it first.

In the last tutorial we created a login and register API as well as the pages needed to interact with them. In this tutorial, we will create the ability for users to create posts. We can do this by creating a new model and then using the CLI to create basic CRUD routes. To do so, lets begin by creating our model:

.. code-block:: bash

	python3 -m libmercury create model post

Once we do this, we can update src/cargo/postModel.py like so:

.. code-block:: python

	from libmercury.db import Column, ForeignKey, Integer, Base, String

	class post(Base):
		__tablename__ = "post"
		id = Column(Integer, primary_key=True)
		title = Column(String(30))
		text = Column(String(300))
		parent_id = Column(ForeignKey("user.id"))

Once we do this, we can create a migration:

.. code-block:: bash

	python3 -m libmercury create migration second_migration

Now we can program our migration in src/migrations/2.py like so:

.. code-block:: python

	from libmercury.db import Column, ForeignKey, Integer, String, MigrationWrapper 
	
	_version = '2'
	_prev_version = None
	
	def upgrade(url):
		wrapper = MigrationWrapper(url)
		wrapper.create_table("post", [
			Column("id", Integer, primary_key=True),
			Column("title", String(30)),
			Column("text", String(300)),
			Column("parent_id", ForeignKey("user.id"))
		])
	
	def downgrade(url):
		wrapper = MigrationWrapper(url)
	
Now let's run our migration like so:

.. code-block:: bash

	python3 -m libmercury migrate

Before we create a CRUD API setup, let's test our model:

.. code-block:: bash

	python3
	Python 3.10.6 (v3.10.6:9c7b4bd164, Aug  1 2022, 17:13:48) [Clang 13.0.0 (clang-1300.0.29.30)] on darwin
	Type "help", "copyright", "credits" or "license" for more information.
	>>> from libmercury import query
	>>> from src.cargo.userModel import user
	>>> from src.cargo.postModel import post
	>>> from src.cargo.connection import Connection
	>>> new_user = user(username="exampleuser", email="test@test.com")
	>>> new_user.set_password("password123") # Very secure password (:
	>>> new_post = post(title="Top 10 National Parks In The US.", text="...", parent_id=new_user.id)
	>>> Connection.Session.add_all([new_post, new_user])
	>>> Connection.Session.commit()
	>>> query(post, title="Top 10 National Parks In The US.").first()
	<src.cargo.postModel.post object at 0x109a68760>
	>>> post_id = query(post, title="Top 10 National Parks In The US.").first().id
	1
	>>> owner = query(user, id=post_id).first()
	<src.cargo.userModel.user object at 0x109a6ad40>
	>>> owner.username
	'exampleuser'

Once we have done this, we can now create our CRUD API like so:

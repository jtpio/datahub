.. _howto/remove-users-orm:

============================
Remove inactive users from hub ORM
============================

JupyterHub performance sometimes scales with the *total* number of users in its
ORM database, rather than the number of running users. Reducing the user count
enables the hub to restart much faster. While this issue should be addressed,
we can work around it by deleting inactive users from the hub database once in
a while. Note that this does not delete the user's storage.

The script `scripts/delete-unused-users.py` will delete anyone who hasn't
registered any activity in a given period of time, double checking to make sure
they aren't active right now. This will require users to log in again the next
time they use the hub, but that is probably fine.

This should be done before the start of each semester, particularly on hubs
with a lot of users. 

Run the script
==============

You can run the script on your own device. The script depends on the `jhub_client` python library. This can be installed with `pip install jhub_client`.

#. You will need to acquire a JupyterHub API token with administrative rights. A hub admin can go to {hub_url}/hub/token to create a new one.
#. Set the environment variable `JUPYTERHUB_API_TOKEN` to the token.
#. Run `python scripts/delete-unused-users.py {hub_url}`

The script currently does not paginate properly, meaning that it operates on the first 200 users provided by the hub. If there are less then 200 active users it is sufficient to keep running the script in a loop until all inactive users are removed. If there are more than 200 active users this procedure will be inadequate. (the script needs to be fixed!)

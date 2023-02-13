HOWTO Canister.io
=================

This HOWTO aims to help users to use `Canister.io <https://canister.io/>`_ to manage the Cloud Container Registry Access provided by Canister.io

Canister.io is a secure and private cloud docker registry for all that includes 20 private repos for free!


Index
-----

#. `HOWTO Manage Docker as a non-root user`_
#. `HOWTO Register an Account on Canister.io`_
#. `HOWTO Create a Docker Repository`_
#. `HOWTO Login`_
#. `HOWTO Create and Push a new Docker image`_
#. `HOWTO Pull a Docker image`_

HOWTO Manage Docker as a non-root user
--------------------------------------

* https://docs.docker.com/engine/install/linux-postinstall/

[`Index`_]


HOWTO Register an account on Canister.io
----------------------------------------

To register a Free account on it you must do the following steps:

#. Sign Up into by submitting your `Canister.io Registration <https://cloud.canister.io/registration>`_

[`Index`_]


HOWTO Create a Docker Repository
--------------------------------

#. `Login <https://canister.io/login>`_ to Canister.io with the account registered (**Solo Account** or **Team Account**).
#. Press on the button ``Create Repo +`` on the right-top corner.
#. Insert, at least, the ``Repository Name`` used to identify the repository where pushing its images and ``submit`` the form to create repo.

[`Index`_]


HOWTO Login
-----------

* ``docker login cloud.canister.io:5000`` and enter the username and password of the account registered.

[`Index`_]


HOWTO Create and Push a new Docker image
----------------------------------------

#. Move into the directory where is the Dockerfile used to create the Docker image
#. Build the Docker image:

   * ``docker build -t <repo-name>:<tag> -f Dockerfile .``

   replace ``<repo-name>`` and ``<tag>`` with the correct value. Suggestion for tags: https://semver.org/
   
   If ``:<tag>`` it is not indicated, will be used ``latest`` value.

#. Tagging the Docker image created:

   * ``docker image tag <repo-name>:<tag> cloud.canister.io:5000/<username>/<repo-name>:<tag>``
   
   replace ``<username>``, ``<repo-name>`` and ``<tag>`` with the correct value.

#. Push the Docker image to Canister.io:

   * ``docker image push cloud.canister.io:5000/<username>/<repo-name>:<tag>``

   replace ``<username>``, ``<repo-name>`` and ``<tag>`` with the correct value.
   
   If ``:<tag>`` it is not indicated, will be used ``latest`` value.
   
[`Index`_]


HOWTO Pull a Docker image
-------------------------

* ``docker pull cloud.canister.io:5000/<username>/<repo-name>:<tag>``

  replace ``<username>``, ``<repo-name>`` and ``<tag>`` with the correct value.
  
  If ``:<tag>`` it is not indicated, will be used ``latest`` value

[`Index`_]

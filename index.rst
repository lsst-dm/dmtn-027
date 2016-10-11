:tocdepth: 1
.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::


.. Add content below. Do not include the document title.

.. note::

   **This technote is not yet published.**

Introduction
============

Say you have a repository named `meas_worst` and you want to change it to better reflect its usage as the best `pipe_tasks` package ever: `pipe_best`. Several other pacakges depend on `meas_worst`, and others may be currently working on `meas_worst` ticket branches. This document describes how to rename the repository with minimal disruption to the LSST build system and its users.

Before you get to this point, be sure you've created an RFC on the topic, gotten agreement about the package rename, and created an implementation ticket in which to do the work.

 NOTE: The document assumes you are part of the `lsst-dm` github organization, but can apply to other groups as well by replacing `lsst-dm` with your organization in the text below.

The process
-----------

Here are the basic steps:

1. Fork
2. Rename package
3. Create branches in old and new package
4. Rename code directories and files in new pacakge
5. Rename ups files
6. Search and replace in source code and docs
7. Update `lsstsw/etc/repos.yaml`
8. Mark old package as deprecated in Readme and via ImportErrors in python
9. Branch dependent packages
10. Update depdendent ups tables and import/include statements
11. Update `confluence package history`_

.. _Confluence package history: https://confluence.lsstcorp.org/display/DM/DM+Stack+Package+History

Post-move cleanup
-----------------

Because github keeps track of forks, we could not move our renamed package back from `lsst-dm` into `lsst`. Once you are reasonably confident that there is nobody working on `meas_worst` and that all relevant work has been moved to `pipe_best`, you can delete `meas_worst` from `lsst` and transfer `pipe_best` from `lsst-dm` to `lsst` via the `github transfer repository`_ instructions. Once you've done the transfer, update `repos.yaml` again. This final rename step should not disrupt anything, since github creates a redirect when you use its repository transfer machanism.

Summary of these steps:

1. Ensure all work has moved to new fork
2. Delete old package
3. Move new package to `lsst` organization
4. Update `lsstsw/etc/repos.yaml`

.. _github transfer repository: https://help.github.com/articles/transferring-a-repository-owned-by-your-organization/

:tocdepth: 1

Introduction
============

Say you have a repository named ``meas_worst`` and you want to change it to better reflect its usage as the best ``pipe_tasks`` package ever: ``pipe_best``. Several other packages depend on ``meas_worst``, and other developers may be currently working on ``meas_worst`` ticket branches. This document describes how to rename the repository with minimal disruption to the LSST build system and its users.

Before you get to this point, be sure you've created an `RFC`_ on the topic, gotten agreement about the package rename, and created an implementation ticket in which to do the work.

NOTE: This document assumes you are part of the `lsst-dm` github organization, but it applies to other groups as well, by replacing `lsst-dm` with your organization in the text below.

.. _RFC: https://developer.lsst.io/processes/decision_process.html#request-for-comments-rfc-process

The process
===========

Here are the basic steps:

1. Fork
2. Rename package
3. Create branches in old and new package
4. Rename code directories and files (**commit**)
5. Rename ups files (**commit**)
6. Search and replace in source code and docs (**commit**)
7. Update :file:`lsstsw/etc/repos.yaml`
8. Deprecate old package
9. Branch dependent packages
10. Update dependent packages
11. Test in Jenkins
12. Update `confluence package history`_ page
13. Get reviewed
14. Post on Community and Slack

We will go through them one at a time in detail below.

.. _Confluence package history: https://confluence.lsstcorp.org/display/DM/DM+Stack+Package+History

Fork
----

Fork ``meas_worst`` from the `lsst` organization into the `lsst-dm` organization via the github interface. When you click the "Fork" button, github should give you a "Where should we fork this repository?" dialog: choose `lsst-dm`. This is the repository we will work in from now on.

Rename package
--------------

Browse to your newly forked ``meas_worst`` repository and click the `Settings` tab. Change the repository name to ``pipe_best`` and click `Rename` (the button will be grayed out until you modify the name).

Create branches in old and new package
--------------------------------------

Create a ``tickets/DM-NNNN`` branch of your `RFC`_ implementation ticket in both the original ``meas_worst`` and the new ``pipe_best``.

Rename code directories and files (**commit**)
----------------------------------------------

Use :command:`git mv` to rename directories and files (in particular: ``python/lsst/meas/worst/`` and ``include/lsst/meas/worst``) so that they fit within the new namespace implied by the new package name. Git is generally quite intelligent about file renames, so you can do, for example::

    cd python
    git mv meas pipe
    cd pipe
    git mv worst best
    git commit -a

By committing this step individually, you will make it much cleaner for others to rebase your changes when they switch to the renamed fork.

Rename ups files (**commit**)
-----------------------------

Rename the files in `ups/`::

    cd ups
    git mv meas_worst.build pipe_best.build
    git mv meas_worst.cfg pipe_best.cfg
    git mv meas_worst.table pipe_best.table
    git commit -a

Search and replace in source code and docs (**commit**)
-------------------------------------------------------

This step is helped immensely by using an editor that can do search-and-replace across all files in a package.

* You will have to change ``meas_worst`` to ``pipe_best`` in the base level :file:`Sconstruct` file, and possibly also in other :file:`Sconscript` files as well.
* In python code, you will need to change ``lsst.meas.worst`` to ``lsst.pipe.best`` and any related ``import lsst.meas.worst as measWorst`` aliases.
* In C++ code and headers, change any ``namespace meas { namespace worst`` blocks to ``namespace pipe { namespace best``, and also change any ``meas::worst`` explicit namespaces.
* Finally, replace any other references to ``meas_worst`` with ``pipe_best``, including docstrings, comments, Config keys and values, and :func:`getPackageDir()` calls. Be thorough in your search.

Commit these changes, and push your branch.

Update ``lsstsw/etc/repos.yaml``
--------------------------------

On your ticket branch, add a ``pipe_best`` entry to lsstsw's :file:`repos.yaml` pointing to `http://github.com/lsst-dm/pipe_best`. Create a Pull Request for your branch, and if it passes Travis you are free to merge it to master (further details on the `Adding New Package`_ page).

Do not delete the ``meas_worst`` entry yet.

.. _Adding New Package: https://developer.lsst.io/build-ci/new_package.html#adding-a-new-package-to-the-build

Deprecate old package
---------------------

On your ticket branch in ``meas_worst``, add a deprecation note to the top of the README file, and add the following to each of the :file:`__init__.py` files in the ``python/meas`` sub-directories::

    raise ImportError("This package is being renamed to pipe_best! Do not use!")

Commit those changes and push your branch.

This will help you identify packages that depend on ``meas_worst``, as they will break as soon as they attempt to import any portion of it.

Branch dependent packages
-------------------------

Create ``tickets/DM-NNNN`` branches for those packages you know are dependent on ``meas_worst``. Any that you weren't aware of will be discovered when you run Jenkins in a moment.

Update dependent packages
-------------------------

In each of the dependent packages:

* Update the ups dependencies in the ``ups/PACKAGE.table`` file: ``setupRequired(meas_worst)`` -> ``setupRequired(pipe_best)`` and in the dependencies section of ``ups/PACKAGE.cfg``.
* Change any python import statements and namespaces, and any C++ include files and namespaces (top-level or explicit).
* Commit and push your branch.

Test in Jenkins
---------------

You can test the individual dependent packages one at a time, but you also need to test everything in Jenkins to ensure you haven't missed any dependent packages. Because you've added ``pipe_best`` to :file:`repos.yaml` on master and done all of the above work on one ticket branch, you can submit a Jenkins job for your branch and it will test all of your changes, plus it will make it clear if you've missed anything because of the ``ImportError`` statements in ``meas_worst``.

Update confluence package history page
--------------------------------------

Add another entry to the `confluence package history`_ table, noting the date you expect the code review and merge for this package rename to be complete.

Get reviewed
------------

Once Jenkins passes, including all the demos, and you've pushed your changes to all of the dependent package branches, have your ticket reviewed. This step should not be too difficult for the reviewer, even though many packages have changed, as the individual changes should be small. You can refer the reviewer to this document for them to refer to during the review (e.g. to prevent file renames and content changes being part of the same commit).

Post on Community and Slack
---------------------------

To ensure that other developers are aware of the pending change, post to the appropriate rooms on Slack (e.g. `dm`) and write up a short Community_ post describing the change and any caveats that other developers should be aware of.

.. _Community: https://community.lsst.org

Merging in work that had started on the old package
===================================================

Once your rename has been merged to master, other developers may have open branches on ``meas_worst`` that they will want to move to ``pipe_best``. Because you did the various steps above as individual commits, they should be able to rebase cleanly. The first step the other developer should do is to clone ``pipe_best`` and check that it has all the changes in their branch:

1. Clone pipe_best: :command:`git clone https://github.com/lsst-dm/pipe_best.git`
2. Checkout the branch: :command:`git checkout tickets/DM-MMMM` and compare it with your work-in-progress branch of ``meas_worst``.

If the branches do not match, (i.e. if the other developer had not pushed all changes to ``meas_worst`` before the fork was created), they will have to follow this procedure to get their latest changes and commits into ``pipe_best``:

1. Commit any changes and push the branch on ``meas_worst`` to github to preserve them.
2. Change the ``meas_worst`` clone's remote: :command:`git remote set-url origin https://github.com/lsst-dm/pipe_best.git`
3. Push the branch to the new remote :command:`git push --set-upstream origin tickets/DM-MMMM`. **Do not pull or push master!** The goal is to update ``pipe_best`` with your branch's changes, but to not change anything else.
4. Update the checked-out branch on the ``pipe_best`` clone created above: :command:`git pull`
5. Check that the branch in the new clone matches your branch in ``meas_worst``, and if so, continue with the steps below. If the branches still don't match, please ask for help from another developer.

If the branch in ``meas_worst`` and the branch in the new clone of ``pipe_best`` contain the same set of changes (this will be the case if the other developer had pushed all of their changes to their branch of ``meas_worst`` before you created the fork, or they had followed the above steps if not), the rest of the merging process is simple:

1. Rebase the new ``pipe_best`` clone to master: :command:`git rebase master`
2. Fix any conflicts. There may be a few, if the branch has modified lines near lines that were changed during the rename.
3. Commit and push the branch to ``pipe_best``, and continue working on your new ``pipe_best`` clone. You can delete your local ``meas_worst`` clone.


Post-move cleanup
=================

Because github keeps track of forks, we could not move our renamed package back from `lsst-dm` into `lsst`. Once you are reasonably confident that there is nobody working on ``meas_worst`` and that all relevant work has been moved to ``pipe_best``, you can delete ``meas_worst`` from `lsst` and transfer ``pipe_best`` from `lsst-dm` to `lsst` via the `github transfer repository`_ instructions. Once you've done the transfer, update `repos.yaml` again, removing the ``meas_worst`` entry and changing the ``pipe_best`` entry to refer to the `lsst` organization. This final rename step should not disrupt anything, since github creates a redirect when you use its repository transfer mechanism.

Summary of these steps:

1. Ensure all work has moved to new fork
2. Delete old package
3. Move new package to `lsst` organization
4. Update `lsstsw/etc/repos.yaml`

.. _github transfer repository: https://help.github.com/articles/transferring-a-repository-owned-by-your-organization/

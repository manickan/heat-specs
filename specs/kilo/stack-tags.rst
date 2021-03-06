..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

..

============
 Stack Tags
============

https://blueprints.launchpad.net/heat/+spec/stack-tags

This feature will allow attributing a set of simple string-based tags
to stacks and optionally the ability to hide stacks with certain tags
by default.

Problem description
===================

Heat should be usable by cloud providers for behind-the-scenes orchestration of
cloud infrastructure, without exposing the user to the resulting
automatically-created stacks.

For example, creation of a Nova server might include, by default, creation and
configuration of a network, subnet, port, and security group.  The "server
create" function in the cloud portal would make a call to Heat instead of Nova.
When the user clicks the "server create" button in the cloud portal, Heat would
then orchestrate the Nova server creation along with calls to other services
and then wire it all up.

Sahara already uses Heat for its internal orchestration, and currently when we
instantiate a OS::Sahara::Cluster resource in a template, the user also sees
the underlying stack created by Sahara.  It would be nice if operators of
Sahara service also could add such specific tags to their internally created
stacks to hide them from common user by default.  That also might concern Trove
when it moves to using Heat orchestration internally.

As other services use heat behind the scenes, they would set specific tags to
such stacks (e.g. source:nova, source:sahara, etc) which, optionally, could be
configured not to be displayed by default, effectively hiding them from regular
users of the API.  Since Heat seems to be no longer a purely user-facing
orchestration service, it makes sense to use these tags as a means to prevent
cluttering of the user's stacks and avoid confusion.

Proposed change
===============

Add a "tag" flag to the stack-create API, which, if given, will create the
stack with such tags.  Also add a configuration option that will allow
operators to hide specific tags from the default stack list.

Add a "show_hidden" flag to the stack-list API, which, if passed, will
result in listing both hidden and non-hidden stacks.  By default, only
non-hidden stacks will be displayed in the stack-list output.

Alternatives
------------

- Using Nova plug-ins for orchestration (not the best tool for the job).

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jasondunsmore

Milestones
----------

Target Milestone for completion:
  Kilo-3

Work Items
----------

- Add a "stack_tag" table.

- Add a "tags" parameter to stack-create (engine and API).  Note: Tag
  names must not contain a comma, as specified in the spec:
  https://review.openstack.org/#/c/155620/

- Add ability to update stack tags during stack-update (engine and
  API).  It should be possible to remove all tags from a stack.

- Add a "show_hidden" parameter to stack-list in engine (engine and
  API).

- Add a "tags" parameter to stack-list in engine (engine and API).
  Passing a tag name will result in only stacks containing that tag
  being shown.  If multiple tags are passed, they will be combined
  using the AND boolean expression.

- Add a "tags-any" parameter to stack-list in engine (engine and API).
  Passing a tag name will result in only stacks containing that tag
  being shown.  If multiple tags are passed, they will be combined
  using the OR boolean expression.

- Add a "not-tags" parameter to stack-list in engine (engine and API).
  Passing a tag name will result in only stacks NOT containing that
  tag being shown.  If multiple tags are passed, they will be combined
  using the AND boolean expression.

- Add a "not-tags-any" parameter to stack-list in engine (engine and
  API).  Passing a tag name will result in only stacks NOT containing
  that tag being shown.  If multiple tags are passed, they will be
  combined using the OR boolean expression.

- Add an API to list tags, ie. "heat tag-list" (engine and API).

- Ensure tags show up in the "heat stack-show <stack>" output (engine
  and API).

- Add docs for new API parameters to "api-site" project.

- Write unit tests to ensure that other stack operations continue to
  work as expected with hidden stacks, eg. stack-show, resource-list,
  stack-list pagination...

- Register a configuration parameter that contains a list of tags to
  hide by default.

- Implement changes to the DB/service/RPC to hide stacks according to
  the configuration parameter.

- Add "show_hidden" parameter to stack-list in python-heatclient.

- Add "--tags", "--tags-any", "--not-tags", and "--not-tags-any"
  options to filter stack-list output by tag in python-heatclient.

Dependencies
============

None.


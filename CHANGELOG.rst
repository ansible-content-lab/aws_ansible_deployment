========================================
Aoc.Controller_Demo_Config Release Notes
========================================

.. contents:: Topics


v1.0.5
======

Release Summary
---------------

Bug fixes

Minor Changes
-------------

- Prevent user defined tags from overwriting persistent tags.

v1.0.4
======

Release Summary
---------------

Updates to tagging.

Minor Changes
-------------

- Added Ansible tags where missing to infrastructure role.
- Updated AWS tagging to allow for user defined tags along with collection tag opinions.

v1.0.3
======

Release Summary
---------------

General updates and bug fixes

Minor Changes
-------------

- Added variable to control removing SSH key from install server regardless of installer outcome.
- README updates.

v1.0.2
======

Release Summary
---------------

Improvements to the install steps to improve experience.

Minor Changes
-------------

- Added a default for `aap_installer_ssh_key_dest` so that it is not required and will default to the ec2-user's ssh directory.
- Added ansible.cfg file with SSH args to keep connection open during AAP install.
- Added output steps to make it easier to find connection details after deployment.
- Updated README to add SSH info.
- Updated get existing VMs logic to make it more reusable.

v1.0.1
======

Release Summary
---------------

Bug fixes and improvements.

Minor Changes
-------------

- Added EDA server public IP address to the list of ALLOWED_HOSTS in EDA.
- Changed include role statements to use FQRN.
- Update task labels to be more descriptive.
- Update the installer unarchive process to avoid errors.
- Updated host group names for clarity.

v1.0.0
======

Release Summary
---------------

Initial release

Major Changes
-------------

- Deploy the RPM-based Ansible Automation Platform installer via RHEL-based virtual machines on AWS.

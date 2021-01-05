AnsibleRole: ansible_scapscan
=========

Run a SCAP scan on a host using specified profile and retrieve reports and score. Report file names are expected at roll-call (see 'Example Playbook' section but default to writing to the host entrypoint directory (generally ~${ANSIBLE_USER}/) and creating scap_scan_results-YYYYMMDD_HHMMSS.xml scap_scan_results_YYYYMMDD_HHMMSS.html.

Currently only supports the following Linux distributions
* Red Hat Enterprise Linux 7.x
* Community Enterprise Linux 7.x

Requirements
------------

This role has no requirements on any additional roles and will install all package dependencies on it's own. If run in a closed environment please check defaults/main.yml scap_required_pkgs.{{ ansible_os_family }} for packages that need to be made available in repositories.

Role Variables
--------------

Below are the default role variables which can be changed when the role is called if needed/desired.

# Default location to write scap html report
scap_html_report: ./scap_scan_results-{{ ansible_date_time.year }}{{ ansible_date_time.month }}{{ ansible_date_time.day }}_{{ ansible_date_time.hour}}{{ansible_date_time.minute }}{{ ansible_date_time.second }}.html

# Default location to write scap xml report (raw results)
scap_xml_report: ./scap_scan_results-{{ ansible_date_time.year }}{{ ansible_date_time.month }}{{ ansible_date_time.day }}_{{ ansible_date_time.hour}}{{ansible_date_time.minute }}{{ ansible_date_time.second }}.xml

# Each distribution has it's own way of being called out in scap, this is simply a lookup to convert from something like RedHat to rhel for SCAP naming
scap_distro_scapname:
  {{ ansible_os_family }}: [value]

# This is the default threshold for which a scan is considered failed. A score >= to this is considered passing. 
scap_score_threshold: 90.00

# Display the scan score at role completion
scap_display_score: true

# Display the stdout of the actual scan
scap_display_scan_run: false

# Path to oscap binary per distribution family
oscap:
  RedHat: /usr/bin/oscap

# Required packages for openscap to run per distribution family
scap_required_pkgs:
  RedHat:
    - openscap-scanner
    - scap-security-guide
    - unzip

# Path to scap content files per distribution family
scap_content_path:
  RedHat: "/usr/share/xml/scap/ssg/content"

# Content file to load per distrubution family/major verison
scap_content_file:
  RedHat: "{{ scap_content_path['RedHat'] }}/ssg-{{ scap_distro_scapname['RedHat'] }}{{ ansible_distribution_major_version }}-ds.xml"

# Available scap profiles per distribution family/major verison
scap_profiles:
  {{ ansible_os_family }}:
    {{ ansible_distribution_major_version }}:
      - title: [ Profile Title ]
        id: [ Profile ID ]

scap_html_report
scap_xml_report

Dependencies
------------

N/A

Example Playbook
----------------

  Sample of calling role from a playbook.
    * scap_profile_title must always be specified, this is the scap profile to scan against
    * scap_html_report and scap_xml_report are recommended but the defaults can be used
    * scap_score_threshold is not required but this shows how the default can be overwritten

  post_tasks are simply shown to give an example on how to access the score results after run but are not required. After a run, scap_score will be available as a variable that represents the scan result score and scap_score_pass will be available as a boolean variable that represent whether the scan score met the threshold or not.

      - name: Run security scans
        hosts: all
        roles:
          - name: ansible_scapscan
            scap_profile_title: "DISA STIG for Red Hat Enterprise Linux 7"
            scap_html_report: "/exports/reports/scap_scan_results-{{ ansible_date_time.year }}{{ ansible_date_time.month }}{{ ansible_date_time.day }}_{{ ansible_date_time.hour}}{{ansible_date_time.minute }}{{ ansible_date_time.second }}.html"
            scap_xml_report: "/exports/reports/scap_scan_results-{{ ansible_date_time.year }}{{ ansible_date_time.month }}{{ ansible_date_time.day }}_{{ ansible_date_time.hour}}{{ansible_date_time.minute }}{{ ansible_date_time.second }}.xml"
            scap_score_threshold: 80.0
        post_tasks:
          - name: Display SCAP score from run
            debug:
              var: scap_score

          - name: Display SCAP Pass from run
            debug:
              var: scap_score_pass

License
-------

BSD

Author Information
------------------

TSW

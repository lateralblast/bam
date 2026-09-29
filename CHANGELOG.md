# Changelog

All notable changes to this project are documented in this file.

## [0.5.4] - 2026-09-29

- Fixed AMT functions to return value like the iDRAC function

## [0.5.3] - 2026-09-29

- Fixed changed status so get functions and non execute runs are not reported as changed

## [0.5.2] - 2026-09-29

- Fixed SSH key auth detection to honour usesshkey and treat any non-empty password as a password

## [0.5.1] - 2026-09-29

- Fixed AMT get function leaving the browser session open on early return

## [0.5.0] - 2026-09-29

- Fixed AMT method default so it is assigned to http rather than compared

## [0.4.9] - 2026-09-29

- Updated AMT code, documentation and requirements for selenium 4.10 or later
- Added missing beautifulsoup4 and lxml to requirements and documentation

## [0.4.8] - 2026-09-29

- Fixed unset URL in AMT set function when no option is given

## [0.4.7] - 2026-09-29

- Fixed AMT power function loading page after selecting power option

## [0.4.6] - 2026-09-29

- Fixed undefined search variable in AMT get function

## [0.4.5] - 2026-09-29

- Added missing time import used by AMT power function

## [0.4.4] - 2026-09-29

- Added missing hostname parameter used by AMT set function

## [0.4.3] - 2023-10-24

- Updated documentation and added handling for text option

## [0.4.2] - 2023-10-24

- Added better handling handling for iDRAC sshpkauth

## [0.4.1] - 2023-10-23

- Improved routine to get value

## [0.4.0] - 2023-10-20

- Bug fixes

## [0.3.9] - 2023-10-20

- Added code to retun value and updated documentation

## [0.3.8] - 2023-10-19

- Bug fixes

## [0.3.7] - 2023-10-07

- Bug fixes and documentation updates

## [0.3.6] - 2023-10-06

- Bug fixes and documentation updates

## [0.3.5] - 2023-10-05

- Bug fixes and documentation updates

## [0.3.4] - 2023-10-04

- Bug fixes

## [0.3.3] - 2023-10-03

- Updated documentation

## [0.3.2] - 2023-10-03

- Updated documentation

## [0.3.1] - 2023-10-02

- Updated documentation

## [0.3.0] - 2023-10-02

- Added and cleaned up more options/tags/switches

## [0.2.9] - 2023-10-02

- Added and cleaned up options/tags/switches

## [0.2.8] - 2023-10-02

- Bug fixes and documentation updates

## [0.2.7] - 2023-10-02

- Added verbose tag

## [0.2.6] - 2023-10-01

- Fixed racadm command output for non execute mode

## [0.2.5] - 2023-10-01

- Added starttime and reboottype flags for jobqueue

## [0.2.4] - 2023-09-30

- Added code to clean up racadm output

## [0.2.3] - 2023-09-30

- Added option to ignore cert warning

## [0.2.2] - 2023-09-30

- Added option to ignore/force cert check

## [0.2.1] - 2023-09-30

- Fixed str/bool config in options

## [0.2.0] - 2023-09-30

- Fixed racadm method

## [0.1.9] - 2023-09-30

- Fixed pexpect import

## [0.1.8] - 2023-09-30

- Fixed iDRAC function call where method is racadm

## [0.1.7] - 2023-09-30

- Fixed AMT set function call

## [0.1.6] - 2023-02-16

- Bug fixes and improvements

## [0.1.5] - 2023-02-12

- Added set support for AMT

## [0.1.4] - 2023-02-12

- Added get support for AMT

## [0.1.3] - 2023-02-10

- Fixes and documentation updates

## [0.1.2] - 2023-02-10

- Added search tag

## [0.1.1] - 2023-02-10

- More bug fixes and improvements

## [0.1.0] - 2023-02-08

- Bug fixes and improvements

## [0.0.9] - 2023-02-08

- Improved output handling

## [0.0.8] - 2023-02-08

- Code cleanup

## [0.0.7] - 2023-02-08

- Collapsed idrac code into more generic code

## [0.0.6] - 2023-02-07

- Added more iDRAC support

## [0.0.5] - 2023-02-06

- Consolidated iDRAC functions into a single function

## [0.0.4] - 2023-02-06

- Modified parameters to allow easier future expansion

## [0.0.3] - 2023-02-06

- Added ability to ssh with key file

## [0.0.2] - 2023-02-03

- Added shell of examples directory

## [0.0.1] - 2023-02-03

- Initial commit

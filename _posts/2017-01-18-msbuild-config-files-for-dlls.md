---
layout: post
title: Using MSBuild for DLL configuration files transformations and output to referencing projects
---

#### What we are going to archive?

Consider folowing setup:

1. ProjectA (Class Library) with custom configuration file foo.config (with configuration transformations)
2. ProjectB (WebProject, ConsoleApplication and so on) that references ProjectA and includes the ProjectA's transformed configuration file foo.config (in output folder of ProjectB)




---
title: "Introduction to Printer Applications"
categories: 
    - OpenPrinting
tags:
    - Printer Application
    - GSoC 2019
---

Hello there, This post is an introduction to my GSoC 2019 project with OpenPrinting, Linux Foundation.

Project Link: [GSoC Website](https://summerofcode.withgoogle.com/projects/#6360869007523840)

### Project Abstract

In classic CUPS-based printing environment, the PPD files and print filters have to be put into standardized directories of the CUPS installation. This method works well in standard RPM or Debian packages. If the CUPS environment is provided in a sandboxed package, adding files to the CUPS installation is not possible. The solution, suggested by Michael Sweet, are Printer Applications. Printer Applications are simple daemons which emulate a driverless IPP network printer on localhost, do the conversion of the print jobs into the printer's format, and send the print job to the printer.

In this project, I aim to implement a universal printer application framework which can be packaged with print filters and PPDs to make up a Printer Application.

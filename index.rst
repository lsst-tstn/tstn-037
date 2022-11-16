:tocdepth: 1

.. sectnum::


Abstract
========

Continuous Integration and Continuous Delivery (CI/CD) are at the core of Telescope and Site Software requirement verification strategy.

The idea relies on continuously deploying and testing (ideally in an automated way) new and existing software with a well defined and understood set of unit and integration tests.
Each new piece of functionality contributes to ultimately fulfilling a high-level requirement, where the verification is continually performed and maintained via the testing frameworks.

Although a large portion of features/requirements can be tested/verified in a CI/CD approach, there are some cases where customized tests are necessary.
This includes, but is not limited to, tests that are directly related to hardware or requirements that are indirectly related to software, e.g. hexapod axis positioning or telescope mount tracking precision.

We also provide artifacts that can be used to certify that the requirements were successfully verified.

Here, we detail the various strategies and tools developed to support software requirement verification for Telescope and Site Software.

Introduction
============

Requirement verification is an important process in the delivery of any project and is what ensures that a project completes its intended deliverables.
In other words, requirement verification provides answers to the question

> Have we built what we intended to build?

Because software is, by definition, extremely dynamic, requirements verification can become quite challenging.
The Telescope and Site Software team, as most software teams in industry at large, utilize an Agile methodology, which means we are continually adapting to focus on the most useful features and delivering new functionality to users at a high cadence.
This also means we are constantly receiving feedback from users and making adjustments to the software based on those feedback.

In terms of requirements verification, the fact that software is constantly changing means we need to be continually verifying requirements.
In order to achieve a constant stream of new features and implement a user-feedback mechanism, we rely on Continuous Integration and Continuous Delivery (CI/CD).

The CI/CD discipline originates from the `Agile methodology <https://www.agilealliance.org/agile101/>`__ as a way to respond to the rapid delivery of new software on a project.
Overall, the idea is that each new software feature (or bug-fixes) must also contains a set of tests that verify the functionality.
In general, tests are separated in two different categories; unit tests and integration tests.

Unit tests are small scale tests designed to verify a particular functionality of a software component.
They usually run in isolation utilizing mocks for other components when required.
Unit tests are the lowest level of testing and, although they usually exercise the requirements, they are not sufficient for requirement verification.
This is often because the are executed in isolated environments or there is no generated report which can be easily evaluated by a reviewer.

Integration tests usually consist of standard or custom operations that exercise a particular component or a group of components in a systematic way.
These tests are designed to run both on testing and production environments.
Because these tests are executed in an environment hosting the integrated system, they provide the perfect conditions to generate requirements verification artifacts.

A CI/CD system is an integral part of Telescope and Site Software deliverables, alongside a suite of integration tests, and a reporting system that are designed to verify the system integrity after every new release and for daily checkouts prior to night-time observations.

We plan to use this suite of integration tests to verify the majority of Telescope and Site Software requirements, alongside any affiliated requirements that can be verified in the process and use the reporting system to generate verification artifacts.
In the following sections we will go over the details of the different steps in the testing process and reporting products.

Testing Suite
=============

In this section, we discuss the collection of tests that are most relevant for the formal requirement verification.
One should be reminded that, in addition to the tests discussed in this section, there are also minimal functionality tests and other categories of tests completed at various stages of the system lifecycle to safeguard the integrity of each component in the system.


SAL Scripts
-----------

SAL Scripts are specialized components designed to execute short lived actions in the observatory control system, including, but not limited to; enabling components, preparing the system for calibrations, taking calibration, tracking targets and taking data, etc.
In principle, most operations executed by the system will be performed using the ScriptQueue, by SAL Scripts.

By default, these components are developed using the BaseScript Python class provided by `salobj`_.
They are developed in one of the two available repositories; `ts_externalscripts`_ and `ts_standardscripts`_, and each change to the code undergoes a peer review process to ensure TSSW development guidelines have been followed.
This includes ensuring that sufficient unit tests are present.
See :tstn:`010` for details on the use case for each of these repositories.

.. _salobj: ts-salobj.lsst.io
.. _ts_externalscripts: https://github.com/lsst-ts/ts_externalscripts
.. _ts_standardscripts: https://github.com/lsst-ts/ts_standardscripts

The usage of SAL Scripts for integration testing comes, mainly, in two different varieties:

#.  Scripts used in standard operations that can be executed in an environment where the definition of success can be explicitly defined (e.g. track_target_and_take_image).
    These can also be used to test fault scenarios by running the script in a mode that we know will cause a particular condition.
    For example, use the ``track_target_and_take_image`` script in a situation that would cause the system to run into a limit while taking data.
#.  A custom made script to test a particular condition.
    For example, a series of hexapod motions designed to test the system against overheating.

In the first case, we simply use a regular operation (e.g. slew and take data) with a particular configuration for which we can easily predict the outcome, whereas scripts designed to execute an integration test have pre-defined predictable behavior.
The first line of testing consists in making sure the script executes successfully, followed by a verification that the data produced by the different components matches our expectations (see :ref:`verification-artifacts`).

The most significant advantage of using standard operational SAL Scripts in integration and verification tests is that we are using the same tools that are regularly used in operations.
Most of these operations exercise the vast majority of the system requirements, which makes them a perfect tool to verify them.
Furthermore, because the majority of the tests utilize TSSW high-level libraries (e.g. ts_observatory_control), the tests are easily maintained, requiring minimal to no updates when there are underlying changes to the CSCs behavior.

Developing specialized SAL Scripts that are only used to verify requirements generally means that the system is being used in a way that does not add capability nor additional functionality to the end-user.
Therefore, unless absolutely necessary, this methodology is discouraged as it results in a significant increase of effort to support the development, testing, and maintenance of the code.
This is discussed further in :ref:`exceptional-cases`.

Jupyter Notebooks
-----------------

Jupyter notebooks provide a complimentary interactive interface into the Rubin Observatory Control system.
They are used in parallel with SAL Scripts to execute tasks that require user interaction or a finer degree of control over the operations.

Although they can be very useful in the early development stages of the process, we do not recommend storing any long-term use software in notebooks.
The primary reason is because it is not currently possible to unit test Jupyter notebooks under the TSSW CI framework, which can lead to bit rot in short timescales, especially with the rapidly evolving software development one would expect in early commissioning stages.
A consequence of this is that any change to the code base potentially breaks the code in the notebook, which could mean that the previous verification is no longer passing.
This is another reason why the use of automated unit and integration testing is a more robust method for requirement verification.

.. _exceptional-cases:

Exceptional Cases
-----------------

.. Since part of Telescope and Site Software scope is to deliver a CI/CD system for the observatory control software, this system

Although a substantial number of requirements can be verified with the use of our CI system, we acknowledge that some of them will inevitably demand a more individualized approach and, sometimes, even a more involved process demanding additional data analysis and/or a more descriptive verification methodology.

Ideally, these cases would be addressed by either developing a customized SAL Script to drive the system through the required test steps or, in the more extreme conditions that might require interactive steps, a Jupyter notebook.
As usual, SAL Scripts are preferred over Jupyter notebooks as they can be unit tested and more easily maintained.
If a test requires a Jupyter notebook, it should be seen as representative of a particular snapshot of the system and there are no guarantees that the test would be reproducible in the future.

Furthermore, any detailed data analysis/methodology should be included in a technical note to be incorporated into the requirement verification artifacts.
It is expected that any documentation produced following a test execution has enough information to allow users to reproduce it in the future, but they should only be seen as representative of a particular snapshot of the system.

.. , since we execute (or plan to) these tests continuously, we can verify that the system continues to fullfil requirements at every iteration.

.. Because these functionalities are (mostly) taken from the requirements pool, it means that each new feature is generally accompanied by a requirement verification test.
.. Nevertheless, tests comes in a variety of different shapes and forms and, most importantly, sometimes it is not possible to fully exercise a given requirement in a testing environment.

.. _verification-artifacts:

Verification Artifacts
======================

In general, successfully executing the testing suite from start to finish gives us confidence that the system is operating as expected.
Nevertheless, the SAL Scripts used during these tests are only designed to fail execution if some critical condition happen (e.g. a component unexpectedly went to FAULT state), and are not designed to verify all the details of every single component involved in the process.

For example, the ``set_summary_state`` SAL Script is designed to execute state transitions and is commonly used in operations to enable components, recover from fault states, etc.
They are executed as an initial step during testing to enable all components and, while they verify the components do transition successfully to the desired state, they **do not** check that the CSCs publish all the required events.
This type of verification needs to be done a posteriori and included in verification artifacts.

These artifacts are human readable reports that contain enough information for a reader with some knowledge about the system can understand what is being verified and how.
For instance, in the example above, the report could show that a particular CSC in the STANDBY state will transition to the DISABLED state after receiving a ``start`` command and will also publish a series of events in the process, with a particular expected payload.
Furthermore, if a component is supposed to publish certain events and fail to do so, the test reports will mark the condition as a failure.
We utilize `Robot Framework`_ to generate, manage and evaluate these reports.

.. _robot-framework:

Robot Framework
---------------

`Robot Framework`_  is a tool designed to assist in test automation and reporting.
In addition to being able to execute tests, it can also be used to generate detailed reports with clear pass/fail criteria.

In our testing suite we mostly rely on Robot Framework to verify the test results and generate test reports.
The tests themselves are driven by a separate service that is in charge of queueing the required SAL Scripts in the ScriptQueue.

An example report generated by Robot Framework is available `here <_static/tts_cycle0026_20220629/report.html>`__.

.. _Robot Framework: https://robotframework.org

One of the key aspects of requirement verification is to be able to map a particular test with a particular requirement it verifies.
For that, Robot framework allows us to add tags to test reports.
Using those tags we can assign the list of requirements that are verified by each individual test.

.. _activity-sequence-diagrams:

Activity Sequence Diagrams
--------------------------

Most of the software requirement verification we perform consists of demonstrating that a specific component executes a particular set of actions.
The success criteria are often represented by the publishing of specific sequence or set of events or telemetry.
In many cases, success is evaluated by observing events and their payloads that are published in response to a command sent to the CSC, or that are published in response to information published by another component.
For example, when a CSC is in the STANDBY state and receives a ``start`` command we expect it to publish a ``summaryState`` event indicating that it transitioned to the DISABLED state.

Activity Sequence Diagrams are broadly used in system design, architecture and systems engineering.
They allow us to specify _how_ a system works by showing the flow of information between components.
As such, they provide a way to demonstrate the sequence of communication during a particular activity, which are useful to support and add context to verification artifacts.

Since all communications between components are stored in the EFD, we can produce Activity Sequence Diagrams for any operation after-the-fact, just like we do with :ref:`Robot Framework <robot-framework>` reports, using a reporting tools specifically designed for this purpose.

The :ref:`figure below <fig-ataos-state-transition>` is an example diagram showing the state transition of the ATAOS CSC from STANDBY to ENABLED.
It shows the commands sent by the SAL Script to the ATAOS CSC, the acknowledgements published by the CSC after receiving the command and the subsequent events (with payload information).

.. figure:: /_static/ataos-state-transition.png
    :name: fig-ataos-state-transition
    :target: ../_images/ataos-state-transition.png
    :alt: ATAOS state transition Activity Diagram

    Activity Sequence Diagram for the transition from STANDBY to DISABLED and subsequently to ENABLED for the ATAOS CSC.

Jira Test Cases
===============

Jira test cases provide a way to organize, plan and track verification test execution.
They can contain detailed information about test execution and provide valuable verification artifacts.

On the other hand, because they are detached from the actual products, they cannot automatically capture changes in the system, which needs to be manually included/updated by a test executor.
Furthermore, executing these test cases alongside the actual test requires dedicate personnel to manually log information, slowing down the process and introducing substantial overhead.

One important role Jira test cases can play in Software Verification is documenting the user interaction with the system, without going into implementation details, and the connection between a particular test and the system requirements.

We also acknowledge that Jira Test Cases can be helpful in planning more complicated verification activities (e.g. those that can not be accomplished by automated testing).
Nevertheless, for the automated test suite we recommend using a simplified test case which contains the minimum information required (e.g. the requirement being verified and an explanation of the test).
The verification artifacts can then contain a link to the :ref:`Robot Framework <robot-framework>` report and possibly an :ref:`Activity Diagram <activity-sequence-diagrams>`, all of which can be generated automatically following the test execution.

Conclusions
===========

Given the substantial scope of Telescope and Site Software and the (inherent) velocity in which software is updated, a sustainable requirements verification strategy requires a considerable level of automation.
Therefore, our approach relies heavily in an automated CI/CD system, alongside reporting tools designed to give high-level overview of the testing results while still allowing us to drill down into the details of each test case.
We expect to be able to cover the majority of the software requirements with this approach, leaving the remaining requirements to be verified by custom test cases.
We recognize that the use of Jira test cases can be helpful in documenting user interaction with the system and provide requirement traceability, but we advise against the inclusion of implementation details.

.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
.. 
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa

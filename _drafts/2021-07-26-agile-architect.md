---
layout: post
title: Value streams from architect perspective
tags:
  - DevOps
aside: true
---
## Flow teams

In agile software development the goal is to organize the people around the value.
Also we need to organize the processes around the people, and with those changes
the organization needs to change, too.

Recently I had opportunity to watch how a hierarchical organization wants to adapt
agile and what issues they had.

### Agile feedback loop

Discovery (learn, experience, try).

## Small agile team

In a development team, requirements are coming as product backlogs, and following
scrum or kanban the development team picks the user stories and implement and test
them. If we are successful, the dev team is able to deploy the code into production,
so we can tick user stories as done by our definition of done.

If not, another team probably the ops team can do the deployment, and after the
successful deployment, we can mark the user story as done. Why do we need to do that?
Because other teams can rely on our features if they are deployed into production,
so other teams can pick up user stories depending on our technical story.

## Multi silo organization

In case of a bigger, non-startup organization, we need to communicate with different
silos, and the flow of the business value is cut by those silos. So if we have
business unit (BU) 'a' which delivers one part of the epic, BU 'b' can start
meaningful work if the BU 'a' finished the work (at least it is finished from the
point of view of the BU 'b'). And BU 'c' can work on the features which has the
technical dependency deployed.

In an ideal world features can be cut into three pieces and one business value can
be delivered sequentially (with some kind of overlap if the technical planning is
good enough). But this is rarely the case. In reality feature 1 can be deployed
easily by BU 'a' but let us say that BU 'b' needs to do a big amount of work to
address its part of the value stream. Sometimes it is not every possible to move
forward without addressing multiple features by BU 'a' in order that BU 'b' can
start working on the features.

### Project management

The million dollar question is: who does the coordination between business units
if we have said that in agile we don't need strong management.

A project manager can coordinate and schedule epic or user story priorities, or
the project manager can do that through a product owner, where the PO understands
the technical dependencies of the solution.

Architects cannot really do project management, from architect standpoint I can
plan the solution from technical point of view and with the help of the PO we
can do the prioritization.

What if BU 'c' has different goals? What that business unit has lower budget to
finish features? I think this is the big challenge of the management to find out
how to prioritize epics between business units. It is no good when different BUs
have different goals, so in theory they can and need to cooperate but in reality
since the priorities are different everyone works by our own.

## Drop the silos

[Effective SD](https://holub.com/heuristics-for-effective-software-development-a-continuously-evolving-list/)

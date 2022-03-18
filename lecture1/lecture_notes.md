# Distributed Systems

## What is it ?

Systems that require more than one Computer systems to get their job done. Mission critical infra. Spread out physically.
> If possible for a system try implementing it on a single computer. Do not go to distributed route unless absolutely needed. Try everything else before distributed systems.

## Why do we need it ?

* Parallelism
* Risk tolerance
* Physical reasons - Commmunication over 2 computers in geographically different zones say.
* Security - Comunication over a narrowly defined network protocol to ensure both systems are safe.
* Isolation

## What challenges we face with them ?

* Different computers working concurrently means concurrency problems which makes Distributed computing hard.
* Multiple pieces + Network means unexpected failure patterns.
* Performance 

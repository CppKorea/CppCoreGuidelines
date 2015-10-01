# Appendix B: Modernizing code

Ideally, we follow all rules in all code.
Realistically, we have to deal with a lot of old code:

* application code written before the guidelines were formuated or known
* libraries written to older/different standards
* code that we just haven't gotten around to modernizing

If we have a million lines of new code, the idea of "just changing it all at once" is typically unrealistic.
Thus, we need a way of gradually modernizing a code base.

Upgrading older code to modern style can be a daunting task.
Often, the old code is both a mess (hard to understand) and working correctly (for the current range of uses).
Typically, the original programmer is not around and test cases incomplete.
The fact that the code is a mess dramatically increases to effort needed to make any change and the risk of introducing errors.
Often messy, old code runs unnecessarily slowly because it requires outdated compilers and cannot take advantage of modern hardware.
In many cases, programs support would be required for major upgrade efforts.

The purpose of modernizing code is to simplify adding new functionality, to ease maintenance, and to increase performance (throughput or latency), and to better utilize modern hardware.
Making code "look pretty" or "follow modern style" are not by themselves reasons for change.
There are risks implied by every change and costs (including the cost of lost opportunities) implied by having an outdated code base.
The cost reductions must outweigh the risks.

But how?

There is no one approach to modernizing code.
How best to do it depends on the code, the pressure for updates, the backgrounds of the developers, and the available tool.
Here are some (very general) ideas:

* The ideal is "just upgrade everything." That gives the most benefits for the shortest total time.
In most circumstances, it is also impossible.
* We could convert a code base module for module, but any rules that affects interfaces (especially ABIs), such as [use `array_view`](#S-GSL), cannot be done on a per-module basis.
* We could convert code "botton up" starting with the rules we estimate will give the greatest benefits and/or the least trouble in a given code base.
* We could start by focusing on the interfaces, e.g., make sure that no resources are lost and no pointer is misused.
This would be a set of changes across the whole code base, but would most likely have huge benefits.

Whichever way you choose, please note that the most advantages come with the highest conformance to the guidelines.
The guidelines are not a random set of unrelated rules where you can srandomly pick and choose with an expectation of success.

We would dearly love to hear about experience and about tools used.
Modernization can be much faster, simpler, and safer when supported with analysis tools and even code transformation tools.
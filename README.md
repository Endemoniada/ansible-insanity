# ansible-insanity
Examples of the worst ways to use Ansible I've come across

---

I came across this mind-bending use of variables, loops and dynamic includes in a project our team inherited. The lead developer and all his colleagues quit before we came onboard, and we were left to untangle this mess with little to no documentation and a vast array of homebrew solutions replicating existing standard solutions.

The use of jinja-templated vars files can possibly be excused by the sheer amount of complexity in the logic going into structuring the data being constructed, but with just a tad more effort, rendering the templates and reading the variables from them could have been done at runtime, in memory, rather than from the filesystem.

Our biggest head-scratcher was the way the variables kept referencing other variables, until the chain abruptly stopped. We had this one var being declared the value of another variable that didn't seem to exist, as well as another variable that _did_ exist, but was seemingly never used, despite the data it contained being exactly the data used in the end, by the tasks actually making the final changes. We figured out what was going on when we looked at the templated files being created, and started widening our scope to including vars and files _outside_ the role in question, in the global namespace, as well.

As I understand it, the culprit is *dynamic include*. While there are elements of static imports in the role, the relevant parts all happen to be dynamic, even the ones not explicitly typed as such (like the `role` and `vars_files` keywords in the main playbook). This causes the variable declarations and assignments to be carried over, deep into the looped role logic, and being executed only when the variable is finally being referenced. What's worse, even though we assign the original variable _the value_ of another variable _just once_ at the start of the playbook (from our, human perspective), that value keeps changing dynamically as the role tasks execute and loop over the list variable, assigning variables new values each time.

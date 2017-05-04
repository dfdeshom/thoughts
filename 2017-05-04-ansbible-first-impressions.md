When it comes to server provisioning, I've only had experience with
Chef. The new place I work in uses ansible and I was tasked with
writing a playbook for deploying and configuring some of our software
there. The whole experience was overall good. My main gripe with
ansible is that it uses `yaml` files for running actions. I think
that's a huge drawback: instead of having the full python language at
your disposal for writing playbooks, you're limited a weird yaml file
format that gets in your way the moment you want something more
complex. On the other hand, I haven't read enough ansible to come
across someone complaining about this decision, so maybe the people
using it don't see this as a problem. Also, python, unlike ruby is not a good fit at all for DSL languages. So I imagine that  the creators had no choice but to use a configuration file format like yaml instead. 

One thing ansible gets right is ansible-galaxy. In Chef, if you want
to use a cookbook, it has to be cloned in your Chef repository. This can
lead to the temptation of editing the cookbook directly, as outlined
here:
https://github.com/dfdeshom/thoughts/blob/master/2016-06-10-chef-anti-patterns.md
 and is somewhat of an anti-pattern. In ansible, you don't have to
 worry about being tempted, you can install external roles in a
 separate folder `ansible-galaxy install --roles-path roles/galaxy -r
 external_roles.yml` on-demand. Ansible being agentless also helps
 here, because only the deployment machine has to worry about having
 the right roles.
 
 The `ansible-lint` tool is also great for people like me using
 ansible for the first time, it forces you to write clearer
 playbooks. 

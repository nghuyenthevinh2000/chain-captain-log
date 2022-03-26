# Building claims module
1. __Must layout what modules to build at the start.__ 

Why so? this is starport related. If still at beginning after starport scaffold chain, it is easy to scaffold another module into chain since go module version is unchanged.

After upgrade to go module, scaffolding another module into chain will produce errors. This error is produced by incompatible go module version between scaffolded module and current chain. Fixing is very gruesome. 

Therefore, it is best to plan out what modules intended to build in the very beginning.

2. __First thing to build: proto__

Proto will determine module messages.

Everything else inside module will based on these messages. 

A worse scenario would be building module first before building Msg proto. After that, if message is incorrect, one will have to build proto again and all module code structure that depends on that incorrect Msg.

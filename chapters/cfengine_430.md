
<!---
Filename: 430-000-Part-Title-0000\_Copying\_Files.md
-->

# Copying Files

\coloredtext{red}{ 430-000-Part-Title-0000\_Copying\_Files.md }


<!---
Filename: 430-100-File\_Copying-1060-Chapter-Title.md
-->

## File Copying

\coloredtext{red}{ 430-100-File\_Copying-1060-Chapter-Title.md }

\begin{codelisting}
\codecaption{430-100-File\_Copying-1070-Local\_copy\_a\_single\_file.cf}
```cfengine3, options: "linenos": true
body file control { inputs => { "$(sys.libdir)/stdlib.cf" }; }

bundle agent main {

  files:

      "/root/etc_group.bak"

        copy_from    => local_cp("/etc/group"),
        comment      => "Demonstrate local file copy";

}
```
\end{codelisting}
<!---                 
Filename: 430-100-File\_Copying-1080-exercise.exr.md
-->

\begin{aside}
\label{aside:exercise_33}
\heading{Create /root/etc_passwd.bak as a backup copy of /etc/passwd}



\end{aside}
\coloredtext{red}{ 430-100-File\_Copying-1080-exercise.exr.md }
\begin{codelisting}
\codecaption{430-100-File\_Copying-1090-Local\_copy\_a\_directory.cf}
```cfengine3, options: "linenos": true
body file control { inputs => { "$(sys.libdir)/stdlib.cf" }; }

bundle agent main
{
  vars:

      "master_location"
        string       => "/var/cfengine/masterfiles";

  files:

      "/var/cfengine/inputs/."

        comment      => "Update policy files cache from master",
        copy_from    => local_cp("$(master_location)"),
        depth_search => recurse("inf");
}
```
\end{codelisting}
<!---                 
Filename: 430-100-File\_Copying-1100-Exercise.exr.md
-->

\begin{aside}
\label{aside:exercise_34}
\heading{Copy /usr/local/sbin/ to /tmp/mirror/}


1. Use CFEngine to make '/tmp/mirror' contain a copy of '/usr/local/sbin'
(Hint: use a files promise with a copy\_from attribute.)

2. Now create a new file in '/usr/local/sbin' and confirm CFEngine will copy it over.

3. Work out how to mirror file removals. (When a file is removed in '/usr/local/sbin', it should disappear in '/tmp/mirror'.)  (Hint: find the appropriate promise attribute in our Agent Promise Attributes summary.)


\end{aside}
\coloredtext{red}{ 430-100-File\_Copying-1100-Exercise.exr.md }
\begin{codelisting}
\codecaption{430-100-File\_Copying-1110-Remote\_copy.cf}
```cfengine3, options: "linenos": true
# This is an example "policy update" policy that promises that
# /var/cfengine/inputs will be a copy of the hub's
# /var/cfengine/masterfiles
#
# The following is what the update policy looked like early on,
# near CFEngine 3.0.
#
# Now the update policy is more sophisticated, with
# tricks to improve performance, although fundamentally, 
# what the update policy does is still: ensure that local directory
# /var/cfengine/inputs is a copy of the hub's /var/cfengine/masterfiles

bundle agent main {

  vars:

      "remote_path" string => "/var/cfengine/masterfiles";
      "remote_server" string => "$(sys.policy_hub)";

  files:

      "/var/cfengine/inputs"

        handle => "update_inputs_dir",
        comment => "Pull down latest policy set",
        perms => m("600"),
        copy_from => remote_cp("$(remote_path)","$(remote_server)"),
        depth_search => recurse("inf"),
        action => immediate;
}

body perms m(mode)
{
        mode  => "$(mode)";
}

body copy_from remote_cp(from,server)
{
        servers     => { "$(server)" };
        source      => "$(from)";
        compare     => "mtime";
        trustkey    => "true";  # trust the server's public key
}

body depth_search recurse(d)
{
        depth => "$(d)";
        xdev  => "true";
}

body action immediate
{
        ifelapsed => "0";
}
```
\end{codelisting}
\begin{codelisting}
\codecaption{430-100-File\_Copying-1120-Remote\_copy\_with\_round\_robin.cf}
```cfengine3, options: "linenos": true
# Use two remote servers, and round-robin between them.
#
# In practice, large sites run policy servers behind SLB's
# (such as F5 Big IP). But you could also do a software
# load-balance, client-side, with CFEngine alone, as this
# policy illustrates.


bundle agent main
{

  classes:
      "heads"
        handle => "flip_a_coin",
        comment => "Generate a class with a 50% probability",
        expression => isgreaterthan(randomint(1,100), 50);


  files:
      "/tmp/test1copy"
        copy_from => round_robin_cp("/var/cfengine/masterfiles/testfile1",
                                    "10.1.1.10",
                                    "10.1.1.12");
}


body copy_from round_robin_cp(from,server1, server2)
{
        source => "$(from)";

    heads::
        servers => { "$(server1)", "$(server2)" };

    !heads::
        servers => { "$(server2)", "$(server1)" };

}
```
\end{codelisting}

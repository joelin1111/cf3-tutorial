Set a custom class based on the existence of a file. 

Example:

```cfengine3
    classes:

       "file_exists"

           expression  =>  fileexists("/etc/site_id") ;
```

Then write another promise that is conditional on the above class.

Run it when the file exists, and when it does not, and observe how CFEngine dynamically configures your server.

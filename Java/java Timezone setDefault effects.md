[](https://stackoverflow.com/posts/9891971/timeline)

	The default timezone setting in java is kind of screwy. by default, if you set the default timezone, it will affect the entire jvm. however, if you are running with a SecurityManager, and the current security context is not allowed to set the default, then the TimeZone.setDefault() method will instead set a _thread local_ value (so any other code running on the same thread will see this value as the default, but the rest of the jvm will be unaffected). i don't think there is a way to set the default just for your "application" unless you can narrow your application to a specific collection of threads (highly unlikely).
# This is not a working setup!!

In order for the php-fpm dockerfile to have access to the index in the project root, you have to specify that context when running docker build.
```bash
docker build . -f .docker/php-fpm/Dockerfile  
```

the first "." is the "context".  This must be run from the root.  This says, give the Dockerfile access to everything from . on (which includes our code).  This seems to be overcomplicating things when you can just keep everything in the root directory and not think about it. 

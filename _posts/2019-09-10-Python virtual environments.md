Virtual environments are helpful to ship your application with all needed dependencies,  
I have been using virtual environments when working with AWS lambda, to start out using them is really simple.  
```
$ pip3 install virtualenv --user
$ mkdir ~/test 
$ virtualenv --no-site-packages  ~/test
$ cd ~/test && source bin/activate
```
Now all packages installed with pip will live in your virtual environment to exit the virtual environment just type:  
```
$ deactivate
```
  
And if you want to move your code to lambda, just zip up the dependencies along.  
```
$ zip -r9   ~/lambda-test.zip ~/test/lib/python3.7/site-packages *
$ zip -g ~/lambda-test.zip your-lambda.py
```

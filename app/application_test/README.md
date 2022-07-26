# Install your framework here

For example: 

1. Remove the application folder ```rm -Rf application```
2. ```docker run -it --rm -v $(pwd)/application:/var/www/html/public -w /var/www/html docker.io/harddocker/app:latest```
3. Install Laravel ```composer create-project laravel/laravel application```
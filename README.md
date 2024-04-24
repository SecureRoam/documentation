# Secure Roam Companion Documentation

Secure Roam Companion is a set of services that improve the security, privacy, and performance of your network inside a "router" like device. These services are free and open-source software that can be installed on a Raspberry Pi or any other Linux-based system.

## Deployment

The Guide is using [mdBook](https://github.com/rust-lang/mdBook) to generate the documentation. To generate the documentation, you need to install `mdBook` on your system. You can install `mdBook` by following the instructions on the [mdBook GitHub page](https://rust-lang.github.io/mdBook/guide/installation.html).

To generate the documentation, run the following command:

```bash
mdbook build
```

This command will generate the documentation in the `book` directory. You can then open the `index.html` file in your web browser to view the documentation.

### Serve the documentation

To serve the documentation locally, run the following command:

```bash
mdbook serve
```

This command will start a web server on `http://localhost:3000` that serves the documentation. You can then open this URL in your web browser to view the documentation.

### Serve the documentation using Nginx

To serve the documentation using Nginx, you need to install Nginx on your system. You can install Nginx by running the following command:

```bash
sudo apt-get install nginx
```

Next, copy the generated documentation to the Nginx web root directory:

```bash
sudo cp -r book /var/www/html/secure-roam-companion
```

Then, create a new Nginx configuration file for the documentation:

```bash
sudo nano /etc/nginx/sites-available/secure-roam-companion
```

Add the following configuration to the file:

```nginx
server {
    listen 80;
    server_name yourhostname.com;

    root /var/www/html/secure-roam-companion;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

Save the file and enable the configuration by creating a symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/secure-roam-companion /etc/nginx/sites-enabled/
```

Finally, restart Nginx to apply the changes:

```bash
sudo systemctl restart nginx
```

## Services

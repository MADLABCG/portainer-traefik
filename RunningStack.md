# Running Applications on our Stack

Alright so our environment is all configured, let me show you now how easy it is to deploy containers to our new Traefik set-up.

This is where the magic happens. It took me 2 minutes in total to find a dockerize app on the internet, deploy it and make it available to the world wide web. Can you believe that? **TWO MINUTES!** If all my deployments were that easy..

Anyway, here are the steps I took.

I pointed the domain I want to use for the app, to the server. For this example I used `express.domain.com`.

After that I google’d “docker express starter” and I found a repo and forked it. Then on the server I switched to the `apps` folder and ran `git clone`

```bash
git clone https://github.com/MADLABCG/simple-html.git ./simple-html
cd simple-html
```

After that it was time to edit the `docker-compose.yml` file of our app:

```yaml
# docker-compose.yml (from the internet repo)
version: "3"
services:
  simple-html:
    container_name: simple-html
    image: nginx
    networks:
      - proxy
    ports:
      - 8023:80
    volumes:
      - ./:/usr/share/nginx/html
    restart: unless-stopped
networks:
  proxy:
    external: true
```

Now you might be wondering, how should I approach this? The first thing we do is remove the `ports` section, as Traefik will take care of this. For the whole Traefik config we only have to add 4 labels:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.docker.network=proxy"
  - "traefik.http.routers.simple-html-secure.entrypoints=websecure"
  - "traefik.http.routers.simple-html-secure.rule=Host(`simple-html.yourdomain.com`)"
```

So what is happening here?

First we enable this container with `enable=true`, then we add it to the `proxy` network. After that we specify the routers and the entrypoints.

Note that this part: `traefik.http.router.app-secure` should have an unique router identification. So make sure you haven’t used that name yet. Let’s say you want to deploy the exact same app on a different domain and container instance, you could use this label: `traefik.http.router.app1-secure`. Just make sure it’s an unique value.

Now the last part that we need to do in the `docker-compose.yml` file is specifying the networks. So the final `docker-compose.yml` file will look like this:

```yaml
# docker-compose.yml
version: "3"
services:
  simple-html:
    container_name: simple-html
    image: nginx
    networks:
      - proxy
    ports:
      - 8023:80
    volumes:
      - ./:/usr/share/nginx/html
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.simple-html-secure.entrypoints=websecure"
      - "traefik.http.routers.simple-html-secure.rule=Host(`simple-html.yourdomain.com`)"
    restart: unless-stopped
networks:
  proxy:
    external: true
```

## Now let’s run our container

```bash
sudo docker-compose up -d
```

That’s all! We literally only added less than 10 lines to our `docker-compose.yml` file and our container is deployed and ready to receive traffic. Awesome right! 👏

Now our new app is also showing up in Portainer:

![Portainer](https://github.com/MADLABCG/portainer-traefik/blob/master/images/portainer.png?raw=true)

Now our new app is also showing up in Traefik:

![Traefik](https://github.com/MADLABCG/portainer-traefik/blob/master/images/traefik.png?raw=true)

Now whenever you want to add a new applications on your server, just repeat the last few steps. Easy as that!

## Conclusion

Let’s recap this.

We have set-up an awesome configuration stack for running and managing multiple docker containers on one server. Deploying new projects will be very easy after this initial set-up.

Something I want to cover in the next post is integrating a basic CI pipeline that connects with our droplet so we can automatically update our containers on a code push to Github. So stay tuned for that!

See you next time!

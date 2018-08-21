# news-aggregator-2

a [Sails v1](https://sailsjs.com) application


## Local Development (With Docker)

- Start the containers: `npm run app:up`
- Stop the containers: `npm run app:down`


### Command-line scripts

- `npm run sources:collect` Collect and sync sources from Feedbin with the application database.
- `npm run posts:collect` Collect the latest posts from Feedbin and save to the application database
- `npm run posts:unfluff` Get the full text, image, tags, and links using [Node Unfluff](https://github.com/ageitgey/node-unfluff).


## Deploying to Hyper.sh

Deploy the database and attach an IP: 

```bash
hyper run -d -v news-db:/data/db --env-file=.env -p 35019:27017 --size s4 --name news-db mongo:4.0

hyper fip attach 123.456.78.90 news-db
```

Build and push the Docker image:

```bash
docker build -t karllhughes/news . && docker push karllhughes/news && hyper pull karllhughes/news
```

Run the collector(s):

```bash
hyper run --rm --env-file=.env --link=news-db --size m1 karllhughes/news node node_modules/.bin/sails run <COLLECTOR_NAME>
```

Set up Hyper.sh cron jobs to automatically run the collectors:

```bash
# Run source collector every 4 hours
hyper cron create --hour=*/4 --minute=0 --env-file=.env --link=news-db --size m1 --name news-sources-cron karllhughes/news node node_modules/.bin/sails run collect-sources

# Run source meta collector every 4 hours
hyper cron create --hour=*/4 --minute=6 --env-file=.env --link=news-db --size m1 --name news-source-meta-cron karllhughes/news node node_modules/.bin/sails run collect-metadata-for-sources

# Run post collector every 1 hour
hyper cron create --hour=* --minute=2 --env-file=.env --link=news-db --size m1 --name news-posts-cron karllhughes/news node node_modules/.bin/sails run collect-posts

# Run post unfluffer every 1 hour
hyper cron create --hour=* --minute=4 --env-file=.env --link=news-db --size m1 --name news-posts-unfluff-cron karllhughes/news node node_modules/.bin/sails run unfluff-posts
```

Run a web instance (optional):

```bash
hyper run -d --env-file=.env --link=news-db:news-db --size s4 --name news-app -p 80:80 karllhughes/news node app.js --prod

hyper fip attach <IP> news-app
```


## License

> Apache 2.0

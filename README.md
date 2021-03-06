# PubSub

##### pubsub is a wrapper around MongoDB change streams (per-Collection event listening) and Bull queues (scheduled jobs with per-collection querying) to support event subscriptions for simple or complex interactions between services

## Installation

Use the package manager [npm](https://docs.npmjs.com/) to install events.

```bash
npm install @postilion/pubsub
```

## Usage

1. Import the PubSub constructor
```javascript
import { PubSub } from '@postilion/pubsub';
```

2. Define subscriptions with a model, operation, handler, filters and queue options
```javascript
[
    {
        name: 'SyncFilingsByTicker',
        model: models.Company,
        operation: Operation.named,
        handler: filingManager.syncSecFilingFeedByTicker,
        filters: [],
        options: {}
    },
    {
        name: 'GetFilingDocumentsForFiling',
        model: models.Filing,
        operation: Operation.create,
        handler: filingManager.getDocumentsForFiling,
        filters: [
            {
                $match: {
                    status: 'unseeded'
                }
            }
        ],
        options: {}
    }
]
```

3. Open a connection with a mongodb client
```javascript
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/db');
```

4. Make sure that your redis instance is running

5. Create a new instance of `PubSub` with `Subscriptions` and `PubSubOptions`
```javascript
const pubsubOptions: PubSubOptions = {
  redis: 'redis://localhost:6379',
  mongodb: 'mongodb://localhost:27017'
}

const pubsub = new PubSub(subscriptions, pubsubOptions);
```

6. Change a document in a collection you've created a subscription for
```javascript
db.filing.insert({
  _id: "5e06528b29734aab3823235d",
  status: "unseeded",
  company: "5e065276aeee4f3833517b6b",
  publishedAt: "2019-02-01T01:22:40.000Z",
  fiscalYearEnd: "1231-01-01T00:00:00.000Z",
  source: "sec",
  type: "10-K",
  refId: "0001018724-19-000004",
  period: "2018-12-31T00:00:00.000Z",
  url: "https://www.sec.gov/Archives/edgar/data/1018724/000101872419000004/0001018724-19-000004-index.htm",
  name: "Form 10-K",
  filedAt: "2019-02-01T00:00:00.000Z",
  acceptedAt: "2019-02-01T04:22:40.000Z",
  __v: 0
})
```

7. The handler attached to each matching subscription should receive a Job that matches the given filters and contains a Document of the `Model` specificed in the `Subscription.model` field
```javascript
{
  id: "cde20f20-28d9-11ea-9735-99b5e82d5a99",
  name: "GetFilingDocumentsForFiling",
  operation: "insert",
  data: Job {
    _id: "5e06528b29734aab3823235d",
    status: "unseeded",
    company: "5e065276aeee4f3833517b6b",
    publishedAt: "2019-02-01T01:22:40.000Z",
    fiscalYearEnd: "1231-01-01T00:00:00.000Z",
    source: "sec",
    type: "10-K",
    refId: "0001018724-19-000004",
    period: "2018-12-31T00:00:00.000Z",
    url: "https://www.sec.gov/Archives/edgar/data/1018724/000101872419000004/0001018724-19-000004-index.htm",
    name: "Form 10-K",
    filedAt: "2019-02-01T00:00:00.000Z",
    acceptedAt: "2019-02-01T04:22:40.000Z",
    __v: 0
  }
}
```

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

## License
[MIT](https://choosealicense.com/licenses/mit/)

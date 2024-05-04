# MongoDB Atlas

### Mục lục

- [Create connect string](#create-connect-string)
- [Find document](#find-document)
- [Insert document](#insert-document)
- [Update documentt](#update-document)
- [Delete document](#delete-document)
- [Count document](#count-document)
- [Retrieve Distinct Values of a Field](#retrieve-distinct-values-of-a-field)
- [Tìm và in số liệu thống kê lưu trữ cho cơ sở dữ liệu](#run-a-command)
- [Watch for Changes](#watch-for-changes)
- [Thực hiện thay đổi hàng loạt giảm độ trễ giữa serve](#perform-bulk-operations)
- [Fundamentals](#fundamentals)

##### Atlas Search

- [Tạo Atlas index](#create-atlas-index)
  - [Ánh xạ động](#dynamic-mappings)
  - [Ánh xạ tĩnh](#static-mappings)
- [Run queries](#run-queries)
  - [Chọn pipeline](#choose-pipeline)
  - [Sử dụng toán tử và collection](#use-operators)
- [Search Options](#search-options)

### Download and install

```sh
mkdir node_quickstart
cd node_quickstart
npm init -y
npm install mongodb@6.5
```

### Create connect string

```js
const { MongoClient } = require("mongodb");

// Replace the uri string with your connection string.
const uri = "<connection string uri>";

const client = new MongoClient(uri);

async function run() {
  try {
    const database = client.db("sample_mflix");
    const movies = database.collection("movies");

    // Query for a movie that has the title 'Back to the Future'
    const query = { title: "Back to the Future" };
    const movie = await movies.findOne(query);

    console.log(movie);
  } finally {
    // Ensures that the client will close when you finish/error
    await client.close();
  }
}
run().catch(console.dir);
```

### Find document

- Tìm 1 document

```js
collection.findOne(query, options);
const query = { title: "The Room" };
const options = {
  // Sort matched documents in descending order by rating
  sort: { "imdb.rating": -1 },
  // Include only the `title` and `imdb` fields in the returned document
  projection: { _id: 0, title: 1, imdb: 1 },
};

collection.findOne(); // return first document
```

- Tìm nhiều document

```js
collection.find(query, options);

const query = { runtime: { $lt: 15 } };
const options = {
  // Sort returned documents in ascending order by title (A->Z)
  sort: { title: 1 },
  // Include only the `title` and `imdb` fields in each returned document
  projection: { _id: 0, title: 1, imdb: 1 },
};
```

### Insert document

- Thêm 1 document

```js
const doc = {
  title: "Record of a Shriveled Datum",
  content: "No bytes, no problem. Just insert a document, in MongoDB",
};
collection.insertOne(doc);
```

- Thêm nhiều document

```js
const result = await collection.insertMany(docs, options);

const docs = [
  { name: "cake", healthy: false },
  { name: "lettuce", healthy: true },
  { name: "donut", healthy: false },
];
// Prevent additional documents from being inserted if one fails
const options = { ordered: true };
```

### Update document

- Cập nhật 1 document

```js
const result = await collection.updateOne(filter, updateDoc, options);

const options = { upsert: true };
// Specify the update to set a value for the plot field
const updateDoc = {
  $set: {
    plot: `A harvest of random numbers, such as: ${Math.random()}`,
  },
};
```

- Cập nhật nhiều document

```js
const result = await collection.updateMany(filter, updateDoc);

const filter = { rated: "G" };
// Create an update document specifying the change to make
const updateDoc = {
  $set: {
    random_review: `After viewing I am ${
      100 * Math.random()
    }% more satisfied with life.`,
  },
};
```

- Thay thế 1 document

```js
const result = await collection.replaceOne(query, replacement);
// Create a query for documents where the title contains "The Cat from"
const query = { title: { $regex: "The Cat from" } };

// Create the document that will replace the existing document
const replacement = {
  title: `The Cat from Sector ${Math.floor(Math.random() * 1000) + 1}`,
};
```

### Delete document

- Xóa 1 document

```js
const result = await collection.deleteOne(query);
const query = { title: "Annie Hall" };
```

- Xóa nhiều document

```js
const result = await collection.deleteMany(query);
const query = { title: { $regex: "Santa" } };
```

### Count document

- Không tham số nhanh hơn có tham số nhưng chỉ ước lượng

```js
const estimate = await collection.estimatedDocumentCount();
```

- Có tham số truy vấn

```js
const countCanada = await collection.countDocuments(query);
const query = { countries: "Canada" };
// Để tăng tốc độ chúng ta sử dụng query là {}
collection.countDocuments({}, { hint: "_id_" });
```

### Retrieve Distinct Values of a Field

```js
const distinctValues = await movies.distinct(fieldName, query);

const fieldName = "year";
// Chỉ định một tài liệu truy vấn tùy chọn để thu hẹp kết quả
const query = { directors: "Barbra Streisand" };
```

### Run a Command

```js
const db = client.db("sample_mflix");
const result = await db.command({
  dbStats: 1,
});
```

- Kết quả

```json
{
  "db": "sample_mflix",
  "collections": 6,
  "views": 0,
  "objects": 67661,
  "avgObjSize": 1796.811161525842,
  "dataSize": 121574040,
  "storageSize": 105435136,
  "totalFreeStorageSize": 0,
  "numExtents": 0,
  "indexes": 10,
  "indexSize": 19599360,
  "indexFreeStorageSize": 0,
  "fileSize": 0,
  "nsSizeMB": 0,
  "ok": 1
}
```

### Watch for Changes

### Perform Bulk Operations

```js
collection.bulkWrite(operations, options?);
```

### Thực hiện 1 transaction

### Fundamentals

- Nén mạng: giúp giảm lượng dữ liệu được truyền qua mạng giữa MongoDB và ứng dụng.
  Có thể chỉ định một hoặc nhiều thuật toán nén, phân tách chúng bằng dấu phẩy.
  Trình điều khiển sẽ chọn thuật toán đầu tiên trong danh sách được phiên bản MongoDB của bạn hỗ trợ.
- Có 3 thuật toán nén:

* `snappy` for Snappy compression

* `zlib` for Zlib compression "zstd" for

* `Zstandard` compression

```js
const uri =
  "mongodb+srv://<user>:<password>@<cluster-url>/?compressors=snappy,zlib";

const client = new MongoClient(uri);

//Khi sử dụng thuật toán nén Snappy hoặc Zstandard phải
npm install --save snappy
npm install --save @mongodb-js/zstd
```

## Atlas Search

1. ### Create atlas index

##### Dynamic mappings

=> không lập được chỉ mục field có "type": "dateFacet" và "type": "numberFacet"

```js
const result = await collection.createSearchIndex(index);
const index = {
  name: "default",
  definition: {
    /* search index definition fields */
    mappings: {
      dynamic: true,
    },
  },
};
```

##### Static mappings

=> hiệu năng tốt hơn

- Kiểu string

```js
const result = await collection.createSearchIndex(index);
const index = {
  name: "string-search-index",
  definition: {
    /* search index definition fields */
    mappings: {
      dynamic: false,
      fields: {
        genres: {
          type: "string",
          analyzer: "<atlas-search-analyzer>",
          searchAnalyzer: "<atlas-search-analyzer>",
          indexOptions: "docs|freqs|positions|offsets",
          store: true|false,
          ignoreAbove: 1, //interger
          multi: {<string-field-definition>},
          norms: "include|omit"
        },
      },
    },
  },
};
```

- Kiểu boolean, dateFacet, objectId, stringFacet, uuid

```js
const result = await collection.createSearchIndex(index);
const index = {
  name: "index-name",
  definition: {
    /* search index definition fields */
    mappings: {
      dynamic: false,
      fields: {
        genres: {
          type: "boolean | dateFacet | objectId | stringFacet | uuid",
        },
      },
    },
  },
};
```

- Kiểu autocomplete

```js
const result = await collection.createSearchIndex(index);
const index = {
  name: "autocomplete-search-index",
  definition: {
    /* search index definition fields */
    mappings: {
      dynamic: false,
      fields: {
        genres: {
          type: "autocomplete",
          analyzer: "lucene-analyzer",
          tokenization: "edgeGram|rightEdgeGram|nGram",
          minGrams: 2,
          maxGrams: 15,
          foldDiacritics: true | false,
        },
      },
    },
  },
};
```

- Kiểu document và document nhúng

```js
const result = await collection.createSearchIndex(index);
const index = {
  name: "document-search-index",
  definition: {
    /* search index definition fields */
    mappings: {
      dynamic: false,
      fields: {
        awards: {
          type: "document | embeddedDocuments",
          dynamic: true | false,
        },
      },
    },
  },
};
```

- Kiểu geo
  ```js
  const result = await collection.createSearchIndex(index);
  const index = {
  name: "geo-search-index",
  definition: {
    /* search index definition fields */
    mappings: {
      dynamic: false,
      fields: {
        awards: {
          indexShapes: "true | false"
          type: "geo",
        },
      },
    },
  },
  };
  ```
- Kiểu knnVector
  ```js
  const result = await collection.createSearchIndex(index);
  const index = {
    name: "knnVector-search-index",
    definition: {
      /* search index definition fields */
      mappings: {
        dynamic: false,
        fields: {
          awards: {
            type: "knnVector",
            dimensions: 4096, // không thể lớn hơn 4096
            similarity: "euclidean | cosine | dotProduct",
          },
        },
      },
    },
  };
  ```
- Kiểu number

```js
const result = await collection.createSearchIndex(index);
const index = {
  name: "number-search-index",
  definition: {
    /* search index definition fields */
    mappings: {
      dynamic: false,
      fields: {
        awards: {
          type: "number",
          representation: "int64|double",
          indexIntegers: true | false,
          indexDoubles: true | false,
        },
      },
    },
  },
};
```

- Kiểu token

```js
const result = await collection.createSearchIndex(index);
const index = {
  name: "number-search-index",
  definition: {
    /* search index definition fields */
    mappings: {
      dynamic: false,
      fields: {
        awards: {
          type: "token",
          normalizer: "lowercase | none",
        },
      },
    },
  },
};
```

### View index

```js
const result = await collection.listSearchIndexes("<index-name>").toArray();
console.log(result);
```

### Edit index

```js
await collection.updateSearchIndex("<index-name>", index);
const index = {
  /* updated search index definition */
  mappings: {
    dynamic: false,
    fields: {
      "<field-name>": {
        type: "<field-type>",
      },
    },
  },
};
```

### Delete index

```js
await collection.dropSearchIndex("<index-name>");
```

2. ### Run queries

### Choose pipeline

`$search` : thực hiện tìm kiếm toàn văn trên trường hoặc các trường được chỉ định. Trường hoặc các trường phải được bao phủ bởi chỉ mục Tìm kiếm Atlas .

- Cú pháp:

```sh
{
  $search: {
    "index": "<index-name>",
    "<operator-name>"|"<collector-name>": {
      <operator-specification>|<collector-specification>
    },
    "highlight": {
      <highlight-options>
    },
    "concurrent": true | false,
    "count": {
      <count-options>
    },
    "searchAfter"|"searchBefore": "<encoded-token>",
    "scoreDetails": true| false,
    "sort": {
      <fields-to-sort>: 1 | -1
    },
    "returnStoredSource": true | false,
    "tracking": {
      <tracking-option>
    }
   }
}
```

- [Details](https://www.mongodb.com/docs/atlas/atlas-search/aggregation-stages/search/)

`searchMeta`: trả về các loại tài liệu kết quả siêu dữ liệu khác nhau.

- Cú pháp

```sh
{
  $searchMeta: {
    "index": "<index-name>",
    "<collector-name>"|"<operator-name>": {
      <collector-specification>|<operator-specification>
    },
    "count": {
      <count-options>
    }
  }
}
```

- [Details](https://www.mongodb.com/docs/atlas/atlas-search/aggregation-stages/searchMeta/)

### Use Operators

1. `autocomplete`:thực hiện tìm kiếm một từ hoặc cụm từ có chứa một chuỗi ký tự từ chuỗi đầu vào không đầy đủ.Các trường truy vấn phải được lập chỉ mục.

- Cú pháp :

```sh
{
  $search: {
    "index": "<index name>", // optional, defaults to "default"
    "autocomplete": {
      "query": "<search-string>",
      "path": "<field-to-search>",
      "tokenOrder": "any|sequential",
      "fuzzy": <options>,
      "score": <options>
    }
  }
}
```

- Ví dụ cơ bản:

* Tạo chỉ mục:

```sh
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "title": [
        {
          "type": "stringFacet"
        },
        {
          "type": "string"
        },
        {
          "foldDiacritics": false,
          "maxGrams": 7,
          "minGrams": 3,
          "tokenization": "edgeGram",
          "type": "autocomplete"
        }
      ]
    }
  }
}
```

- Truy vấn:

```js
const result = await collection.aggregate(agg);

const agg = [
  { $search: { autocomplete: { query: "off", path: "title" } } },
  { $limit: 10 },
  { $project: { _id: 0, title: 1 } },
];
```

- Ví dụ mờ:

```js
const result = await collection.aggregate(agg);

const agg = [
  {
    $search: {
      autocomplete: {
        query: "pre",
        path: "title",
        fuzzy: { maxEdits: 1, prefixLength: 1, maxExpansions: 256 },
      },
    },
  },
  { $limit: 10 },
  { $project: { _id: 0, title: 1 } },
];
```

- Ví dụ về thứ tự mã thông báo:

```js
const result = await collection.aggregate(agg);
const agg = [
  {
    $search: {
      autocomplete: { query: "men with", path: "title", tokenOrder: "any" },
    },
  },
  { $limit: 4 },
  { $project: { _id: 0, title: 1 } },
];
```

- Ví dụ nổi bật:

```js
const result = collection.aggregate(agg);

const agg = [
  {
    $search: {
      autocomplete: { query: "ger", path: "title" },
      highlight: { path: "title" },
    },
  },
  { $limit: 5 },
  {
    $project: {
      score: { $meta: "searchScore" },
      _id: 0,
      title: 1,
      highlights: { $meta: "searchHighlights" },
    },
  },
];
```

- Tìm kiếm trên nhiều lĩnh vực:

```js
const result = await collection.aggregate(agg);
const agg = [
  {
    $search: {
      compound: {
        should: [
          {
            autocomplete: {
              query: "inter",
              path: "title",
            },
          },
          {
            autocomplete: {
              query: "inter",
              path: "plot",
            },
          },
        ],
        minimumShouldMatch: 1,
      },
    },
  },
  {
    $limit: 10,
  },
  {
    $project: {
      _id: 0,
      title: 1,
      plot: 1,
    },
  },
];
```

- Kết quả nhóm thông qua truy vấn thuộc tính:

```js
const result = collection.aggregate(agg);

const agg = [
  {
    $searchMeta: {
      facet: {
        operator: {
          autocomplete: { path: "title", query: "Gravity" },
        },
        facets: {
          titleFacet: { type: "string", path: "title" },
        },
      },
    },
  },
];
```

- [Details](https://www.mongodb.com/docs/atlas/atlas-search/autocomplete/)

2. `compound`: kết hợp hai hoặc nhiều toán tử vào một truy vấn duy nhất.
   - Cú pháp

```sh
{
  $search: {
    "index": <index name>, // optional, defaults to "default"
    "compound": {
      <must | mustNot | should | filter>: [ { <clauses> } ],
      "score": <options>
    }
  }
}
```

- [Các toán tử khác](https://www.mongodb.com/docs/atlas/atlas-search/operators-and-collectors/)

### Search Options

- [Sort](https://www.mongodb.com/docs/atlas/atlas-search/sort/)
- [Parallelize Query Execution Across Segments](https://www.mongodb.com/docs/atlas/atlas-search/concurrent-query/)
- [Highlight Search Terms in Results](https://www.mongodb.com/docs/atlas/atlas-search/highlighting/)
- [Count Atlas Search Results](https://www.mongodb.com/docs/atlas/atlas-search/counting/)
- [Paginate the Results Sequentially](https://www.mongodb.com/docs/atlas/atlas-search/paginate-results/)
- [Track Search Terms](https://www.mongodb.com/docs/atlas/atlas-search/tracking/)
- [Return Stored Source Fields](https://www.mongodb.com/docs/atlas/atlas-search/return-stored-source/)

iso-bmff-parser-stream
======================

Parse an ISO BMFF using nodejs.

Returns a structured javascript object of iso-bmff boxes.


What is ISO-BMFF: [tldr version](http://en.wikipedia.org/wiki/ISO_base_media_file_format), [full version](http://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=61988)

**This is still a work in progress !**

## Installation

```
npm install iso-bmff
```

## How to use

### Parse a media file

```
var chunkFile = './media/audio.m4s'

var fs = require('fs');
var isoBmff = require('../index.js');

var chunkStream = fs.createReadStream(chunkFile, {
	flags: 'r',
	encoding: null,
	fd: null,
	mode: 0666,
	autoClose: true
});

var unboxing = new isoBmff.Parser(function (err, data) {
	console.dir(JSON.stringify(data));
})


chunkStream
	.pipe(unboxing);

```

Output is something like this

```
[  
   {  
      "id":1,
      "type":"styp"
   },
   {  
      "id":2,
      "type":"sidx",
      "content":{  
         "version":0,
         "flags":0,
         "referenceId":1,
         "timeScale":12800,
         "earliestPresentationTime":2517504,
         "firstOffset":0,
         "entries":[  
            {  
               "referencedSize":815756,
               "subSegmentDuration":25600
            },
            {  
               "referencedSize":117691,
               "subSegmentDuration":5120
            }
         ]
      }
   },
   {  
      "id":3,
      "type":"moof",
      "content":[  
         {  
            "id":4,
            "type":"mfhd"
         },
         {  
            "id":5,
            "type":"traf",
            "content":[  
               {  
                  "id":6,
                  "type":"tfhd"
               },
               {  
                  "id":7,
                  "type":"tfdt",
                  "content":{  
                     "version":0,
                     "flags":0,
                     "baseMediaDecodeTime":2517504
                  }
               },
               {  
                  "id":8,
                  "type":"trun"
               }
            ]
         }
      ]
   },
   {  
      "id":9,
      "type":"mdat",
      "content":"BINARY_DATA"
   },
   {  
      "id":10,
      "type":"moof",
      "content":[  
         {  
            "id":11,
            "type":"mfhd"
         },
         {  
            "id":12,
            "type":"traf",
            "content":[  
               {  
                  "id":13,
                  "type":"tfhd"
               },
               {  
                  "id":14,
                  "type":"tfdt",
                  "content":{  
                     "version":0,
                     "flags":0,
                     "baseMediaDecodeTime":2543104
                  }
               },
               {  
                  "id":15,
                  "type":"trun"
               }
            ]
         }
      ]
   },
   {  
      "id":16,
      "type":"mdat",
      "content":"BINARY_DATA"
   }
]
```

### Build a media file from generated output

`iso-bmff` can re-generate media files, from it's output. This is handy, if you want to modify something in a media file, and generate it again

```
var chunkFile = './media/video.m4s'

var fs = require('fs');
var isoBmff = require('iso-bmff');

var chunkStream = fs.createReadStream(chunkFile, {
	flags: 'r',
	encoding: null,
	fd: null,
	mode: 0666,
	autoClose: true
});

var unboxing = new isoBmff.Parser(parseDone);

chunkStream
	.pipe(unboxing);

function parseDone (err, data) {
	new isoBmff.Builder(data, buildDone);
}

function buildDone (err, data) {
	fs.writeFile(chunkFile.replace('media/', ''), data);
}
```

## Extend with boxes

Every box can have it's own parser module, 
to analyze and parse box binary data, and return something meaningful. Every box has at least two methods, one for **parse** and one for **rebuild**, so box data parse/build logic can be contained in small modules.

See the [lib/box](https://github.com/necccc/iso-bmff-parser-stream/tree/master/lib/box) folder for these modules, and currently supported box types.
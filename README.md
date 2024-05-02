## Decentralized Link Directory - DLD

DLD is a protocol used to store links in a decentralized way on the Bitcoin blockchain using OP_RETURN transactions via a simple scheme.

### Motivation

The motivation behind DLD is to create a decentralized link directory that is censorship resistant and can be used by anyone to store or retrive links. In the current world, censorship is a big problem and many countries are blocking access to certain websites. DLD can be used to store links to these websites in a decentralized way so that they can be accessed by anyone.

### How it works

By using a simple scheme we can allow anyone to broadcast links and their intent.

The scheme has 3 parts each separated by a colon `:`
1. Prefix - `DLD:`
2. Version - `1` (current version)
3. Operation code
    -  `0` - Add link
    -  `1` - Upvote link
    -  `2` - Downvote link
    - If operation code is `0` then the link should be provided
        - The parser only cares about the domain name `subdomain+domain` or `domain`. If the protcol is included, it will be stripped. So no need to use up precious space. For example, use `galaxoidlabs.com` instead of `https://galaxoidlabs.com`.
    - Important to note that the if there are more than one link with the same domain name the parser will simply retry to verify. Only one link per domain name is allowed.

The resulting string would be hex encoded and stored in an OP_RETURN transaction. 

Below is an example of a non encoded DLD link that adds a link to `galaxoidlabs.com`

```
DLD:1:0:galaxoidlabs.com
```

And the encoded version of the above which would be put in the OP_RETURN transaction:

```
444C443A313A303A67616C61786F69646C6162732E636F6D
```

Lets break it down visually:

```
DLD:1:0:galaxoidlabs.com
|   | | └-- Since the operation code is 0, this is the link that is to be stored  
|   | └------ Operation code. 0 for add link, 1 for upvote link, 2 for downvote
|   └---------- Version number used to identify the DLD version
└---------------- This is the prefix used to identify DLD links
```

- The first portion of the link is the prefix. This is used to identify DLD links. The prefix is `DLD:`.
- The second portion of the link is the version number. This is used to identify the DLD version. The current version is `1`.
- The third portion of the link is the operation code. This is used to identify the operation to be performed. The operation codes are as follows:
  - `0` - Add link
  - `1` - Upvote link
  - `2` - Downvote link
- The fourth portion of the link is the actual link. This is the link that is to be stored. The link should be a valid URL and should start with `http://` or `https://`. We will describe what should be available for verification in the link below.

### Link verification

In terms of verification my original thought was that we required the trxid of the addition to be stored in the dld.json thats required to be stored at `/.well_known/dld.json` on the domain, but I think that the requirement of having to have a json file on the domain is enough to verify intent. If you didn't want the link aggregated by the dld parser you wouldnt have the dld.json file on the domain. In fact, this is how you would remove a link from the dld parser. You would simply remove the dld.json file from the domain and when the parser runs and sees that the file is missing after a certain amount of iterations it would remove the link from the database. 

This brings us to the json file that is required to be stored on the domain. This file is required to be stored on the domain so that we can verify that the link is actually intended to be stored. It also acts as a way for the DLD parser to know what links are available at the root site.

A json file should stored at: `<link>/.well_known/dld.json`. This file should contain the following:

```json
{
    "version": 1,
    "name": "Galaxoid Labs", // required
    "desc": "Galaxoid Labs is a software development company", // required
    "img": "https://galaxoidlabs.com/img/logo.png", // optional
    "links": [ // At least one link is required
        {
            "url": "https://galaxoidlabs.com", // Basic web link
            "type": "website",
        },
        {
            "url": "https://twitter.com/galaxoidlabs/.well_known/nostr.json", // NIP05 Nostr json file
            "type": "nostr"
        },
        {
            "url": "https://twitter.com/galaxoidlabs/feed.json", // Any url to resolves to your json feed
            "type": "feed_json"
        },
        {
            "url": "https://twitter.com/galaxoidlabs/rss.xml", // Any url to resolves to your rss feed
            "type": "feed_rss"
        },
        {
            "url": "https://twitter.com/galaxoidlabs/atom.xml", // Any url to resolves to your atom feed
            "type": "feed_atom"
        }
    ]
}
```

---
layout: post
title:  "Arc de Triomphe"
date:   2026-07-14 11:29:23 -0400
---

If you look at the Wikidata page for the [Arc de Triomphe](https://www.wikidata.org/wiki/Q64436), in the `Statements` section, the first property is `instance of` (P31).  There are three entries:

- triumphal arch
- tourist attraction
- museum

Presumably most people would interpret this as meaning "it is all of these", because logically these options aren't exclusive, it **can be** all of these things at once.

But in the RDF Wikidata dump, there are *four* statements for P31:

{% highlight json %}
{
  "@id": "wikidata:Q64436",
  "wdp:direct/P31": {
    "@id": "wikidata:Q143912"
  },
  "wdp:P31": [
    {
      "@id": "wikidata:statement/Q64436-84bc9b4f-437c-3930-d216-16209a0bd8c2"
    },{
      "@id": "wikidata:statement/Q64436-5436dbee-4cc8-e425-1cc7-720e138d5ab9"
    },{
      "@id": "wikidata:statement/Q64436-42276ff4-43c7-9fd3-9e40-9c4f7a8e1da4"
    }
  ]
}
{% endhighlight %}
*(simplified JSON-LD)*

Only the first (the "direct") value has a valid Wikidata object URI (the link for "triumphal arch"), maybe because it's the only statement with a reference?  For that, `stated in` (P248) appears on the web page, linking to [Base Mérimée](https://www.wikidata.org/wiki/Q809830).

The other three are internal Wikidata "statements" -- what do those look like? 

{% highlight json %}
{
  "@id": "wikidata:Q64436",
  "wdp:P31": [
    {
      "@id": "wikidata:statement/Q64436-84bc9b4f-437c-3930-d216-16209a0bd8c2",
      "@type": [
        "wbo:Statement",
        "wbo:BestRank"
      ],
      "wbo:rank": {
        "@id": "wbo:PreferredRank"
      },
      "http://www.w3.org/ns/prov#wasDerivedFrom": {
        "@id": "http://www.wikidata.org/reference/d709174cc609072fbf310bb52ddfb7aa2332b473"
      },
      "wdp:statement/P31": {
        "@id": "wikidata:Q143912"
      }
    },
    {
        "@id": "wikidata:statement/Q64436-5436dbee-4cc8-e425-1cc7-720e138d5ab9",
        "@type": "wbo:Statement",
        "wbo:rank": {
          "@id": "wbo:NormalRank"
        },
        "wdp:statement/P31": {
          "@id": "wikidata:Q570116"
        }
    },
    {
        "@id": "wikidata:statement/Q64436-42276ff4-43c7-9fd3-9e40-9c4f7a8e1da4",
        "@type": "wbo:Statement",
        "wbo:rank": {
          "@id": "wbo:NormalRank"
        },
        "wdp:statement/P31": {
          "@id": "wikidata:Q33506"
        }
    }
}
{% endhighlight %}
*(simplified JSON-LD)*

These are the same three P31 values as the web page, but with some additonal "ranking" information and, yes, provenance data for the referenced statement. So, we can infer that the `PreferredRank` statement qualifies to become a "direct" P31 relationship, while the other two do not.

I think this is wrong, or at the very least, misleading.  It breaks logic with what is presented on the web page (all three statements), and it breaks logic with what one would reasonably expect from an RDF dump; ie. everything.  Including the statement ranking information is useful, but suppressing statements is not.

More practically, the class for which *Arc de Triomphe* belongs ("triumphal arch") is also, obviously and by far, the smallest.  This is a problem for [Wiki Core](https://wikicore.ca) which clusters items partly based on the size of the classes that they're in -- niche classes like "triumphal arch" are currently underrepresented.

The [WikiProject Ontology Group](https://www.wikidata.org/wiki/Wikidata:WikiProject_Ontology) is responsible for managing how [P31 instance of](https://www.wikidata.org/wiki/Property:P31) is used.  I worry that examples like this reinforce a tree-based ontological view on Wikidata that's not an accurate representation of reality.

The *Arc de Triomphe* is unquestionably a triumphal arch. But it is also a tourist attraction and a museum.
# Compute Cosine Similarity Of Vector Embeddings

The [`pgvector`](https://github.com/pgvector/pgvector) extension for PostgreSQL
allows for storing and querying against vector embeddings. One form of
embeddings I find myself working with are [text embeddings generated with models
on AWS Bedrock](/llm/generate-text-embeddings-with-aws-bedrock.md). Given a
bunch of rows of text and their associated embeddings, I can use vector queries,
like _cosine distance_ to measure how similar or dissimilar one text is from
another. This is useful if I want to find related text entries -- e.g. related
blog posts or [TILs](https://github.com/jbranchaud/til).

The `<=>` operator computes the _cosine distance_ between two embeddings.

```sql
> select tpe1.embedding <=> tpe2.embedding
  from til_post_embeddings tpe1, til_post_embeddings tpe2
  where tpe1.id = 2 and tpe2.id = 1601;

      ?column?
--------------------
 0.4439706636658506
(1 row)
```

The cosine distance of two vectors is always a value between 0 and 2. The
smaller the value (closer to 0) the more similar the two vectors are. The
greater the number (closer to 2) the more dissimilar or opposed they are.

```
┌───────┬────────────┬──────────┬────────────────────────┐
│ angle │ similarity │ distance │        meaning         │
├───────┼────────────┼──────────┼────────────────────────┤
│ 0°    │ 1          │ 0        │ same direction         │
├───────┼────────────┼──────────┼────────────────────────┤
│ 90°   │ 0          │ 1        │ unrelated (orthogonal) │
├───────┼────────────┼──────────┼────────────────────────┤
│ 180°  │ -1         │ 2        │ opposite directions    │
└───────┴────────────┴──────────┴────────────────────────┘
```

The _distance_ can be translated into _similarity_ by subtracting it from `1`.

```sql
> select 1 - (tpe1.embedding <=> tpe2.embedding) as "Cosine Similarity"
  from til_post_embeddings tpe1, til_post_embeddings tpe2
  where tpe1.id = 2 and tpe2.id = 1601;

 Cosine Similarity
--------------------
 0.5560293363341494
(1 row)
```

If I do this for all other posts relative to a given post (`id = 2`) and then
order by that similarity value, then I've surfaced the most similar posts.

```sql
> select til1.id, til2.id, 1 - (tpe1.embedding <=> tpe2.embedding)
  from til_posts til1
  join til_posts til2
    on til1.id != til2.id
  join til_post_embeddings tpe1
    on til1.id = tpe1.til_post_id
  join til_post_embeddings tpe2
    on til2.id = tpe2.til_post_id
  where til1.id = 2
  order by 3 desc
  limit 10;

 id |  id  |      ?column?
----+------+---------------------
  2 | 1601 |  0.5560293363341494
  2 | 1663 | 0.47633089054616984
  2 | 1662 |  0.4131859068031881
  2 |    3 |  0.3704925587920118
  2 |  793 | 0.36239932887556314
  2 | 1672 |  0.3581576836658229
  2 | 1710 |  0.3242143295939479
  2 | 1578 |  0.3140966091780846
  2 |  760 | 0.30534146146003804
  2 | 1627 |  0.2931870309468221
(10 rows)
```

I'm working on using this and a couple other approaches to measuring similar 
posts so that I can add a "Related TILs" section to the bottom of each of my TIL
posts.

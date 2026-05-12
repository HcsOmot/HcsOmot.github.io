---
layout: post
title:  "Migrating data from MariaDB to AWS Redshift"
date:   2026-05-12 20:04:42 +0200
categories: ['software-design', 'databases']
---
While working at ch, I had to migrate some data from Mariadb to AWS Redshift to be used for reporting and BI. As these things usually go, the Redshift tables were already designed and set up by someone else. 
Data source was our production Mariadb table which had a `varchar(32)` field and target was a Redshift table which had a column with a `varchar(16)` field.

For reasons I won't go into, one of the requirements was to use `left(<field>, 16)` and whoever set up the Redshift table beforehand received the same instruction - we were only supposed to migrate the first 16 characters from source. 

As I was told to do, I did. Naively. I tested the query and the whole script using some sample data I had lying around and it worked.
Things seemed simple. Things seemed to work. Task was simple. Ticket was done.

Things unexpectedly broke a few days later and it took me twice as long to figure it out and fix it than it did to create the bug in the first place.
I just got a few sparse logs saying that the input data was too large for the target field and script terminated.


I went digging through Mariadb and Redshift docs to figure out what was going on and found the following:
[Redshift: Character_types-varchar-or-character-varying](https://docs.aws.amazon.com/redshift/latest/dg/r_Character_types.html#r_Character_types-varchar-or-character-varying)

[Redshift: Varchar & bytes](https://docs.aws.amazon.com/redshift/latest/dg/r_Character_types.html#r_Character_types-varchar-or-character-varying:~:text=Use%20a%20VARCHAR%20or%20CHARACTER%20VARYING%20column%20to%20store%20variable%2Dlength%20strings%20with%20a%20fixed%20limit.%20These%20strings%20are%20not%20padded%20with%20blanks%2C%20so%20a%20VARCHAR(120)%20column%20consists%20of%20a%20maximum%20of%20120%20single%2Dbyte%20characters%2C%2060%20two%2Dbyte%20characters%2C%2040%20three%2Dbyte%20characters%2C%20or%2030%20four%2Dbyte%20characters).

My initial assumption was wrong, clearly - Redshift's varchar essentially imposes byte-based constraints on value size, while Mariadb uses character count to achieve the same effect.

[MariaDB: Varchar](https://mariadb.com/docs/server/reference/data-types/string-data-types/varchar)
[MariaDB: Varchar & char count](https://mariadb.com/docs/server/reference/data-types/string-data-types/varchar#:~:text=A%20variable%2Dlength%20string.%20M%20represents%20the%20maximum%20column%20length%20in%20characters)

In addition to that, the `left` command also returns characters - as clearly stated by the docs: [SQL: `left`](https://mariadb.com/docs/server/reference/sql-functions/string-functions/left#:~:text=Return%20the%20leftmost%20characters.%20This%20function%20returns%20the%20specified%20number%20of%20characters%20from%20the%20beginning%20(left)%20of%20a%20string).

So basically, while I got back 16 characters, I also got 16+ bytes of data which Redshift refused at insert time.
While debugging I found that our source data had a few gems that were multibyte. Things like `Letecké sportov ní a servisní centrum Aeroklubu České republiky z.s.`

Event though that string has 67 characters, it also has 72 bytes of data. And the first 16 characters hold 17 bytes of data.

Since this written in PHP, the insert queries were using prepared statements and they were built dynamically by code. And that meant that I could manipulate the values much easier that using SQL.

The fix was relatively simple - using `mb_strcut` to trim down the data to max 16 bytes. I tried `substr` and `mb_substr` but those didn't work correctly since they return the substring and round up to the whole multibyte character, respectively, which also caused overflow.

`mb_strcut`, on the other hand, operates at byte level instead of character level.

I use PHP, btw.
And Mariadb, and occasionally Redshift. If I can't help it.
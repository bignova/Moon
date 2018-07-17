---
layout: post
title: "MySQL Encoding Issue (Latin-1 to UTF-8)"
date: 2018-07-17
excerpt: "Switching MySQL encoding format from Latin-1 to UTF-8"
tags: [Debug]
comments: true
---

Recently, I work on a Flask Web App with some CRUD manipulations. A MySQL ORM, called <a href="https://http://flask-sqlalchemy.pocoo.org/2.3/">Flask-SQLAlchemy</a> is used to create MySQL models and interact with databases. However, the problem happended when I was trying to create a new entry with some Chinese characters in its columns.

<figure>
	<img src="https://user-images.githubusercontent.com/11435445/42801720-1f72627e-89d3-11e8-9f2c-1304da2e7d90.png" alt="MySQL Encoding Error">
    <i>Not the exact error output I have. But it's pretty similar</i>
</figure>


After searching for a few similar problems, I found the issue is the encoding format of MySQL database.

The default encoding format of MySQL is actually Latin-1. To allow characters in other languages, like Chinese characters, we have to switch it to UTF-8.

Therefore, after we crate a new database, the first thing we should do normally is to change its encoding configs to UTF-8. For example:

{% highlight sql %}
set character_set_client=utf8;
set character_set_results=utf8;
set character_set_connection=utf8;
set character_set_database=utf8;
{% endhighlight %}

<figure>
	<img src="https://user-images.githubusercontent.com/11435445/42802437-8f7ef558-89d5-11e8-952a-382ecfdb6058.png" alt="MySQL Encoding Error">
</figure>

However, this is not the whole story. If we check one of the tables in this database, we could find that the configs are like this:

<figure>
	<img src="https://user-images.githubusercontent.com/11435445/42802586-f76bb3ae-89d5-11e8-832a-a8fca8319950.png" alt="MySQL Encoding Error">
</figure>

See, the encoding format of this table is still Latin-1. So what we could know is, those tables which are created before we switch the encoding format of the database would not be changed. So we have to use the following sentence to modify the encoding format of the table:

{% highlight sql %}
alter table BETWEENNESS character set utf8;
{% endhighlight %}

Unfortunately, the error is raised once again after this change. Checking the table again, we notice that even though the table now uses UTF-8 for encoding, some columns are still in Latin-1. 

<figure>
	<img src="https://user-images.githubusercontent.com/11435445/42804158-af721d0e-89da-11e8-9ace-1086b9363a87.png" alt="MySQL Encoding Error">
</figure>

We use the following sentences fix the issue:

{% highlight sql %}
alter database matrix character set utf8;
alter table BETWEENNESS change semantic semantic varchar(32) character set utf8 not null;
alter table BETWEENNESS character set utf8;
{% endhighlight %}

<figure>
	<img src="https://user-images.githubusercontent.com/11435445/42803405-6d0758b4-89d8-11e8-90d0-33c44a657f7f.png" alt="MySQL Encoding Error">
</figure>

This time, there is no more Latin-1 shows up in our table. The issue is eventually solved.

In conclusion, other than changing those default configs of the database, the following two sentences could help you to modify the tables and columns that are already created and set with some other encoding formats:

{% highlight sql %}
alter table <TABLE> character set utf8;
alter table <TABLE> change <COLUMN> <COLUMN> <TYPE> character set utf8;
{% endhighlight %}

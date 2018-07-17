---
layout: post
title: "MySQL Encoding Issue (utf8)"
date: 2018-07-17
excerpt: "Switching MySQL encoding format to UTF-8 to allow chinese characters"
tags: [Debug]
comments: true
---

Recently, I work on a Flask Web App with some CRUD manipulations. A MySQL ORM, called <a href="https://http://flask-sqlalchemy.pocoo.org/2.3/">Flask-SQLAlchemy</a> is used to create MySQL models and interact with databases. However, the problem happended when I was trying to create a new entry with some Chinese characters in its columns.

<figure>
	<img src="https://user-images.githubusercontent.com/11435445/42801720-1f72627e-89d3-11e8-9f2c-1304da2e7d90.png" alt="MySQL Encoding Error">
</figure>
<b><i>Not the exact error output I have. But it's pretty similar</i></b>

After searching for a few similar problems, I found the issue is the encoding format of MySQL databse.

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

See, the encoding format of this table is still Latin-1. So what we could know is those tables that are created before we switch the encoding format of the database would not be changed. So we have to use the following sentence modify the encoding format of the table:

{% highlight sql %}
alter table [TABLE_NAME] character set utf8;
{% endhighlight %}


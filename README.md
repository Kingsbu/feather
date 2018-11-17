# Feather - a fast-developing web server framework


Feather is a modern c++ web framework for rapid development. Feather's goal is to allow users to develop a web site with minimal effort and cost.

Nowadays, many web frameworks are very large and costly to learn. Feather is just trying to solve these problems. It is as light as its name, and everything is designed to make it very convenient and fast for users to develop rather than fall into Among the details of the frame.

A good framework should be to allow users to easily implement their ideas rather than become slaves to the framework. If you want to develop a web framework quickly and easily without having to spend a lot of effort to learn the details of the framework, then Feather is perfect for you! 


## What is Feather?

As a rapid development framework, Feather's goal is to make web server development easy. It mainly has the following characteristics:

1. Simple and easy to use
2. Hich performance, modern c++（c++17）development
3. header only
4. Cross-platform
5. Support for compile-time reflection
6. Support AOP

The core of the Feather framework includes:

1. An easy-to-use http library that supports http1.x, https, websocket
2. A powerful html template engine
3. An extensible ORM library that supports multiple databases (mysql, postgresql, sqlite)
4. An extensible serialization library that supports multiple formats (json, xml) 

## Feather's architecture

Here is the architecture diagram of Feather:

![Feather的架构图](https://github.com/qicosmos/feather/blob/master/framework.png)：

1. http component：  cinatra
2. ORM component：   ormpp
3. Serialization component： iguana
4. html template： inja

The four core components inside Feather are loosely coupled, and Feather just puts them together like building blocks.

## How to use Feather

A simple example shows how to use Feather, such as displaying a list of articles, which is available on almost every website.

The function of getting a list of articles is very simple. The underlying data part is an article table. The business logic layer is to get these lists and then do html rendering. The external interface part is an http service, and the data format of the front-end back-end interaction is json. For the sake of simplicity, caches and the like are not considered.

So how do you do this with Feather to implement this article list function? You can follow these steps to develop:

1. Provide an http interface to get a list of articles;；
2. Provide database access through ORM;；
3. Write business logic, get a list of articles based on user requests and render through html template;

Let's implement the three steps above

### Get the http interface of the article list

	const int max_thread_num = 4;
	http_server server(max_thread_num);
	server.listen("0.0.0.0", "http");

	article_controller article_ctl;
	server.set_http_handler<GET, POST>("/get_article_list", &article_controller::get_article_list, &article_ctl);

The login interface is like this:

	class article_controller{
	public:
		void get_article_list(const cinatra::request& req, cinatra::response& res){
            //todo
            res.set_status_and_content(status_type::ok);
        }
	};
Next, you can test the http service interface, and the client only needs to send an http request. For example, sending a request like this http://127.0.0.1/get_article_list server will be automatically routed to the article_controller::get_article_list function, if the request is not correct, it will return an http error to the client. When the server receives such a request, it indicates that the http service provided by the server is available.

Next, you need to write the code of the database part. Thanks to the ORM, you can easily write the code of the database part, which is also very simple.


### Provide database access through ORM
Login business involves a user table, so we need to create this table, but before you create the database to determine what database you choose, Feather's ORM supports mysql, postgresql and sqlite three databases, assuming our database is mysql. We can create a user table with the following code.

1.Create an article table

	struct article{
        int id;
        std::string title;
        std::string introduce;
        int user_id;
        int visible;
        std::string create_time;
    };
    REFLECTION(article, id, title, introduce, user_id, visible, create_time);

	dbng<mysql> mysql;
	mysql.connect("127.0.0.1", "root", "12345", "testdb")
	mysql.create_table<article>(ormpp_auto_key{ ID(article::id) });

Dao.create_table< article > will automatically create an article table in the testdb database, where the id field is self-growth.

2. Write the logic to get the list of articles (including access to the database)

	void get_article_list(const cinatra::request& req, cinatra::response& res){
		auto page_s = req.get_query_value("page");
        int current_page = atoi(page_s.data());

        dao_t<dbng<mysql>> dao;
        std::vector<article> v;
        dao.get_object(v);
            
        article_list_res list{std::move(v), current_page, total_page(v.size())};

        iguana::string_stream ss;
        iguana::json::to_json(ss, list);

        res.add_header("Access-Control-Allow-origin", "*");
		res.set_status_and_content(status_type::ok, ss.str());
    }

Access the database, serialize it to json and return it to the client.

For a detailed example, you can look <a href="https://translate.googleusercontent.com/translate_c?depth=1&amp;hl=en&amp;prev=search&amp;rurl=translate.google.com&amp;sl=zh-CN&amp;sp=nmt4&amp;u=https://github.com/qicosmos/feather&amp;xid=17259,15700022,15700124,15700149,15700186,15700191,15700201&amp;usg=ALkJrhg4lESHy4YwXTsjZ9-9uEj-U6umhg">at the code on github</a> 

## Demo example

We developed a community website with Feather, here：http://purecpp.org/

Feather Community：
![Feather社区](https://github.com/qicosmos/feather/blob/master/demo.png)

## benchmark
ab -c50 -n3000 xxxurl

![feather benchmark](https://github.com/qicosmos/feather/blob/master/qps.png)

## Acknowledgement

The Feather community website was developed by me and the netizen [XMH](https://github.com/xmh0511/) (it took two days of spare time), XMH is also a loyal user of cinatra, not only contributed a lot of code, but also provided a lot of valuable suggestions, I would like to express my heartfelt thanks!

XMH is a program that loves programming. It is usually engaged in c++, web, mobile and other development. Engaged in game background and APP development. Also a loyal mordern c++ fan, following the development of cpp, like to do some small tools through metaprogramming. I love the open source community, and I am also a user of the open source project. I hope that Feather can be used by more developers, and the community is getting better and better.

I hope more people can join in to make Feather better.

## Contact us

purecpp@163.com

[http://purecpp.org/](http://purecpp.org/ "purecpp")

[https://github.com/qicosmos/feather](https://github.com/qicosmos/feather "feather")

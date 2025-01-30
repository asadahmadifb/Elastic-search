# Elastic-search

    docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" -e "ELASTIC_PASSWORD=your_password" elasticsearch:8.0.0

    https://localhost:9200/
    
    powershell
    [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("elastic:Aa@123456"))

    PUT https://localhost:9200/my_index
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    Content-Type: application/json
    
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      }
    }
    ###
    GET https://localhost:9200/_cat/indices?v
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    
    ###
    POST https://localhost:9200/my_index/_doc/2
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    Content-Type: application/json
    
    {
      "title": "2 Document",
      "filed3": "This is the content of the 2 document."
    }
    
    ###
    GET https://localhost:9200/my_index/_search
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    Content-Type: application/json
    
    {
      "query": {
        "match": {
          "title": "first"
        }
      }
    }
    
    ###
    GET https://localhost:9200/my_index/_search
    Authorization: Basic ZWxhc3RpYzpBYUAxMjM0NTY=
    Content-Type: application/json
    
    {
      "query": {
        "match_all": {}
      }
    }

    --------------------------------------------------------------
    Install-Package NEST

    var settings = new ConnectionSettings(new Uri("http://localhost:9200"))
    .DefaultIndex("your_index_name");
    var client = new ElasticClient(settings);

    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public decimal Price { get; set; }
    }

    var createIndexResponse = client.Indices.Create("products", c => c
    .Map<Product>(m => m
        .AutoMap()
        )
    );

    var product = new Product { Id = 1, Name = "Sample Product", Price = 10.99M };
    var indexResponse = client.IndexDocument(product);

    var searchResponse = client.Search<Product>(s => s
    .Query(q => q
        .Match(m => m
            .Field(f => f.Name)
            .Query("Sample")
            )
        )
    );


   ----------------------------------------
   public class Comment
    {
        public string User { get; set; }
        public string Message { get; set; }
        public DateTime Date { get; set; }
    }
    
    public class Instructor
    {
        public string Name { get; set; }
        public List<Comment> Comments { get; set; }
    }
    
    public class Course
    {
        public string CourseName { get; set; }
        public List<Instructor> Instructors { get; set; }
    }
    
    // تعریف مپینگ
    var createIndexResponse = client.Indices.Create("courses", c => c
        .Map<Course>(m => m
            .Properties(ps => ps
                .Text(t => t
                    .Name(n => n.CourseName)
                )
                .Nested<Instructor>(n => n
                    .Name(nn => nn.Instructors)
                    .Properties(pn => pn
                        .Keyword(k => k.Name(n => n.Name))
                        .Nested<Comment>(cn => cn
                            .Name(nc => nc.Comments)
                            .Properties(pc => pc
                                .Keyword(k => k.Name(c => c.User))
                                .Text(t => t.Name(c => c.Message))
                                .Date(d => d.Name(c => c.Date))
                            )
                        )
                    )
                )
            )
        )
    );
    
    
    var searchResponse = client.Search<Course>(s => s
        .Query(q => q
            .Nested(n => n
                .Path(p => p.Instructors)
                .Query(nq => nq
                    .Bool(b => b
                        .Must(
                            mq => mq.Match(m => m.Field(f => f.Instructors.First().Name).Query("Alice")),
                            nq => nq.Nested(nn => nn
                                .Path(pn => pn.Instructors.First().Comments)
                                .Query(nq2 => nq2
                                    .Match(m => m.Field(f => f.Instructors.First().Comments.First().Message).Query("Great"))
                                )
                            )
                        )
                    )
                )
            )
        )
    );

        
        1.	استفاده از Bulk API
        2.	پیکربندی مپینگ‌های سفارشی
        3.	استفاده از فیلترها
        4.	استفاده از Aggregations
        5.	پیکربندی ریپلیکیشن و شاردینگ
        6.	استفاده از Scripted Fields
        7.	پیاده‌سازی Paging و Sorting
        8.	استفاده از Snippets
        9.	مدیریت خطاها و Retry Policies
    -------------------------------- اضافه کردن چندین سند به صورت همزمان-----------------------------
    var products = new List<Product>
    {
        new Product { Id = 1, Name = "Product 1", Price = 10.99M },
        new Product { Id = 2, Name = "Product 2", Price = 20.99M },
        new Product { Id = 3, Name = "Product 3", Price = 30.99M }
    };
    
    var bulkResponse = client.Bulk(b => b
        .Index("products")
        .IndexMany(products)
    );
    ----------------------------به‌روزرسانی و اضافه کردن اسناد---------------------------------
    var bulkResponse = client.Bulk(b => b
        .Index("products")
        .Update<Product>(u => u
            .Id("1")
            .Doc(new Product { Price = 15.99M }) // به‌روزرسانی قیمت
        )
        .IndexDocument(new Product { Id = 4, Name = "Product 4", Price = 40.99M }) // اضافه کردن سند جدید
    );
    ----------------استفاده از عملیات شرطی--------------------------------------------------
    var bulkResponse = client.Bulk(b => b
    .Index("products")
    .Update<Product>(u => u
        .Id("3")
        .Doc(new Product { Price = 35.99M }) // به‌روزرسانی قیمت
        .Upsert(new Product { Id = 3, Name = "Product 3", Price = 35.99M }) // اگر وجود نداشت، اضافه کن
        )
    );
            --------------------------------مپینگ داده ----------------------------------------------
        var createIndexResponse = client.Indices.Create("products", c => c
            .Map<Product>(m => m
                .Properties(p => p
                    .Text(t => t
                        .Name(n => n.Name)
                        .Analyzer("standard")
                    )
                    .Number(n => n
                        .Name(n => n.Price)
                        .Type(NumberType.Double)
                    )
                )
            )
        );
        -----------------بازیابی کل داده ها --------------------------------------------------
        var searchResponse = client.Search<Product>(s => s
        .Query(q => q
            .MatchAll() // بازیابی تمام اسناد
            )
        );
        ------------------------------------------------------------------------------------------
        var searchResponse = client.Search<Product>(s => s
        .Query(q => q
            .Bool(b => b
                .Must(m => m
                    .Match(mq => mq
                        .Field(f => f.Name)
                        .Query("Product A") // جستجوی دقیق نام محصول
                    )
                )
                .Should(sh => sh
                    .Match(mq => mq
                        .Field(f => f.Description)
                        .Query("special offer") // جستجوی قسمتی از متن در توضیحات
                    )
                )
                .Filter(f => f
                    .Bool(bf => bf
                        .Must(
                            mf => mf.Term(t => t.Category, "Electronics"), // فیلتر بر اساس دسته‌بندی
                            mf => mf.Range(r => r
                                .Field(p => p.Price)
                                .GreaterThanOrEquals(100) // قیمت بیشتر یا مساوی 100
                                .LessThan(500) // قیمت کمتر از 500
                            )
                        )
                    )
                )
            )
        )
    );
    ----------------------------------------------------------------------------------
    .Match: جستجوی متنی و تطابق نزدیک. مناسب برای فیلدهای متنی.
    .Term: جستجوی دقیق و تطابق کامل. مناسب برای فیلدهای غیرمتنی (keyword).

#MongoDB笔记


##MongoDB的安装注意事项
1. Windows安装mongodb前先安装Visual C++的开发环境。
2. 在mongodb的安装目录下新建\data\db、data\log目录文件夹分别存放数据和日志
3. cmd下进入安装目录下运行mongod --dbpath D:\MongoDB\data\db
4. 再次打开cmd命令模式进入安装目录下运行mongo，启动成功,运行数据库

##基础数据
post = {

	"user":"xudongmin",
	"age":12,
	"sex":"man",
	"cousins":[
		{
			"name":"joe",
			"age":14,
			"sex":"woman"},
		{
			"name":"Mary",
			"age":14,
			"sex":"woman"	
		}
	]
} ;
##插入(insert)
* db.user.insert(post);
##读取(find)
* db.user.find();
* db.user.findOne({"user":"xudongmin"});

 第一个参数为搜索条件，第二个参数指定返回的键1/0(返回/去除)

db.user.find({"user":"xudongmin","age":12},{"_id":0});

db.user.find({"user":"xudongmin","age":12},{"_id":1});

* 第二个参数$slice

指定返回哪些键

db.user.find({},{"$slices":20})//查询第21条数据
db.user.find({},{"$slices":[10,20]})//查询第11到第30条数据

##删除(remove)
* db.user.remove({"user":"xudongmin"});
##更新(update)
* db.user.update({"user":"xudongmin"},post);
* unsert(update第三个参数)

如果没有文档符合更新条件，就会以这个条件和更新的数据为基础创建一个新的文档

db.users.update({"count":5},{"$inc":{"count":10}},true);//第三个参数true表示这个是unsert

* update第四个参数

更新所有匹配的文档

db.users.update({"user":"xudongmin"},{"$set":{"count":10}},false,true);

result = db.runLastCommand({getLastError:1})//result.n更新的文档数

##保存(save)

save可以再文档不存在时插入，文档存在时更新

db.user.save(post);

##条件句

* $lt < 、$lte <= 、$gt > 、$gte >=

db.user.find({"age":{"$gt":10,"$lt":30}});

* $in $nin $or

$in、$nin一个键的or查询，$in表示在这个条件组内，$nin表示不再这个条件组内

$or多个键的条件查询

db.user.find({

	"$or":[
			{
				"user":{
					"$in":[
						"xudongmin",
						"Jhon"
					]
				}
			},{
				"age":{
					"$gt":10,
					"$lt":30
				}
			}
		]
});

* $mod $not

$mod会将查询的值除以第一个给定值，若余数等于第二个给定值则返回该结果

$not返回不再给定的值内的结果

db.user.find({"age":{"$not":{"$mod":[5,2]}}});

* $exists

判定键是否存在

db.user.find({"z":{"$exists":true}});//查询存在z键的数据

* $all

通过多个元素来匹配键值

db.user.find({"cousins.name":{"$all":["joe","Mary"]}})；

* $size

获取数组的长度

db.user.find({"cousins":{"$size":2}});//获取cousins键数组长度为2的数据

* $elemMatch

让内嵌文档能够使用条件句的方法

db.user.find({"cousins":{"$elemMatch":{"name":"joe","age":{"$gte":14}}}})

* $where

尽量避免使用$where查询，因为他们在速度上要比常规查询慢很多

db.user.find({"$where":"this.user=='xudongmin'"});

db.user.find({

	"$where":function(){
		return this.user=='xudongmin'; 
	}
});


##修改器
* $inc

修改器$inc可以对文档的某个值为数字型（只能为满足要求的数字）的键进行增减的操作

db.user.update({

	"user":"xudongmin"
	},{
		"$inc":{
			"age":1
		}
});//age加1

* $set

修改器$set直接更改文档的某个存在的值，不存在就增加

db.user.update({

	"user":"xudongmin"
	},{
	"$set":{
		"favorite":"war and peace"
	}
});//增加favorite字段

* $unset
 
修改器$unset 主要是用来删除键,不论对目标键使用1、0、-1或者具体的字符串等都是可以删除该目标键

db.user.update({

	"user":"xudongmin"
	},{
		"$set":{
			"favorite":0
		}
});

* $push

像已有的数组末尾加入一个元素，要是元素不存在，就会创建一个新的元素。

db.user.update({

	"user":"xudongmin"
	},{
		"$push":{
			"cousins":{
				"name":"Jhon",
				"age":15,
				"sex":"man"
			}
		}
});

* $ne

	1. 一个值不在数组里时就把它加进去，避免重复数据 

		db.user.update({
			
			"cousin.name":{
				"$ne":"Ken"	
			},{
				"$push":{
					"cousin.name":"Ken"
				}
			}
		
		});//如果Ken不在数组cousin.name里时，把Ken加入到cousin.name中。

	2. 类似不等于

		* db.users.find({"user":{"$ne":"refactor1"}})//查出所有name不等refactor1的文档,注意 文档中不存在键name的文档也会被查出来

* $addToSet

往数组里面加入数据，如果数组里已经存在，则不会加入（避免重复）

将”addToSet"和"each”组合起来，可以添加多个不同的值，二用”ne"和"push”组合就不能实现。

db.user.update({

	"user":"xudongmin"
	},{
		"$addToSet":{
			"cousins":{
				"$each":[
					{
					"name":"Jhon",
					"age":14,
					"sex":"woman"
					},{
					"name":"Jhon",
					"age":14,
					"sex":"woman"
					}
				]	
 			}
		}
});

* $pop

删除数组元素，只能从头部或尾部删除一个元素 -1从头部删除，1从尾部删除

db.user.update({

	"user":"xudongmin"
	},{
		"$pop":{
			"key":-1
		}
});

* $pull

删除数组元素，将所有匹配的元素删除。

db.user.update({

	"user":"xudongmin"
	},{
		"$pull":{
			"cousins":{
				"name":"Jhon",
				"age":14,
				"sex":"woman"	
 			}
		}
});

* $

数组定位修改器，匹配数组中第一个匹配的元素。

db.user.update({
	
	"cousins.name":"xudongmin"
	},{
	"$set": {"cousins.$.name":liuxing}
})


##findAndModify
db.runCommand({

	"findAndModify":{},
	"query":{},
    "update":{},
    "remove":true|false,
    "new":true|false,
    "sort":{},
    "fields":{},
    "upsert":true|false
});

findAndModify是集合名

query是查询选择器，与findOne的查询选择器相同

update是要更新的值，不能与remove同时出现

remove表示删除符合query条件的文档，不能与update同时出现

new为true：返回个性后的文档，false：返回个性前的，默认是false

sort：排序条件，与sort函数的参数一致。

fields:投影操作，与find*的第二个参数一致。

upsert:与update的upsert参数一样。

##总结

1.  条件语句是内层文档的键，修改器是外层文档的键

	* db.user.find(age:{"$lt":30,"$gt":20});

2.  一个键可以应用多个条件，但是一个键不能对应多个更新修改器

	* db.user.update("$inc":{"age":1},"$set":{"age":30});//错误

3. 查询内嵌文档要求整个文档完全匹配

	*  db.user.find({"cousins":{"name":"joe","age":{"$gte":14}}})//查询不到数据
	*  db.user.find({"cousins":{"name":"joe","age":14}})//能查到数据


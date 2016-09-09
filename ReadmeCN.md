介绍：

Reds 是一个轻量的基于NodeJS和Redis的搜索引擎，由TJ Holowaychuk 开发，这个模块原本是为优化 Kue 搜索能力而开发，但是它也非常适合作为轻量的通用搜索库而加入到Blog、文档系统中去。

安装：

    $ npm install reds

例子：

首先你可能想做的事情就是创建一个可以通过关键字来进行搜索的搜索实例，你可以使用Redis的命名空间以支持在同一个DB里进行多重搜索。

    var search = reds.createSearch(‘pets‘);

Reds严格区分任意的数字和字符串ids，所以你可以用这个库来做任何原本想做的事情，甚至是结构化数据存储，下面的例子仅仅是一个包括了一些字符串数据库数组，我们调用Search#index()来将它加入到Reds.

    var strs = [];
    strs.push(‘Tobi wants four dollars‘);
    strs.push(‘Tobi only wants $4‘);
    strs.push(‘Loki is really fat‘);
    strs.push(‘Loki, Jane, and Tobi are ferrets‘);
    strs.push(‘Manny is a cat‘);
    strs.push(‘Luna is a cat‘);
    strs.push(‘Mustachio is a cat‘);
     
    strs.forEach(function(str, i){ search.index(str, i); });

执行一个操作以防止Redsg来作为字符串调用Search#query()，然后通过回调函数来获取一个执行完成之后包含ids或者空数组的返回结果。

    search
      .query(query = ‘Tobi dollars‘)
      .end(function(err, ids){
        if (err) throw err;
        console.log(‘Search results for "%s":‘, query);
        ids.forEach(function(id){
          console.log(‘  – %s‘, strs[id]);
        });
        process.exit();
      });

当Reds匹配到搜索的关键词，将输出如下的结果：

    Search results for "Tobi dollars":
      – Tobi wants four dollars

我们通过reds.search()也可以用Reds在回调函数里进行联合查询。

    search
      .query(query = ‘tobi dollars‘)
      .end(function(err, ids){
        if (err) throw err;
        console.log(‘Search results for "%s":‘, query);
        ids.forEach(function(id){
          console.log(‘  – %s‘, strs[id]);
        });
        process.exit();
      }, ‘or‘);

如果匹配到包含：”Tobi”和”dollars”关键词的语句时，将返回如下结果：

    Search results for "tobi dollars":
      – Tobi wants four dollars
      – Tobi only wants $4
      – Loki, Jane, and Tobi are ferrets

API

    reds.createSearch(key)
    Search#index(text, id[, fn])
    Search#remove(id[, fn]);
    Search#query(text, fn[, type]);

例如：

    var search = reds.createSearch(‘misc‘);
    search.index(‘Foo bar baz‘, ‘abc‘);
    search.index(‘Foo bar‘, ‘bcd‘);
    search.remove(‘bcd‘);
    search.query(‘foo bar‘).end(function(err, ids){});


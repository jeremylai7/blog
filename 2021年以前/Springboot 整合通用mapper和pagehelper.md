## 简介
### springboot
设计目的就是为了加速开发，减少xml的配置。如果你不想写配置文件只需要在配置文件添加相对应的配置就能快速的启动的程序。

### 通用mapp
通用mapper只支持对单表的操作，对单表的增删改查，无需在mapper.xml写对应的sql语句,只需要我们调用相应的接口即可。
### pagehelp
pagehelper主要是在对查询的数据进行一个分页查询。
1. 首先在maven项目，在pom.xml中引入mapper和pagehelper的依赖
```
        <!-- pagehelp -->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper-spring-boot-starter</artifactId>
			<version>1.2.3</version>
		</dependency>
		<!-- 通用mapper -->
		<dependency>
			<groupId>tk.mybatis</groupId>
			<artifactId>mapper-spring-boot-starter</artifactId>
			<version>1.0.0</version>
		</dependency>
```
2 新建一个mymapper.java文件，继承mapper接口
```
public interface MyMapper<T> extends Mapper<T>, MySqlMapper<T>,ConditionMapper<T> {
  //FIXME 特别注意，该接口不能被扫描到，否则会出错
}
```    
这个java文件不能和其它mapper放在一起，以免被扫描到。获取单表数据的操作都直接调用这个方法。

3 在配置文件上添加以后属性字段
```
#jdbc
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/news
spring.datasource.username=数据库用户名
spring.datasource.password=数据库密码
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.freemarker.request-context-attribute=request

#mapper  
mapper.mappers=com.imooc.springboot.mapper.util.MyMapper
mapper.not-empty=false
mapper.identity=MYSQL

#pagehelper
pagehelper.helper-dialect = mysql
pagehelper.reasonable = true
pagehelper.support-methods-arguments = true
pagehelper.params= count= countSql
```
上面的配置mapper.mappers 是第2步里面文件所在的路径。

4  添加了controller文件之后，由controller里面的方法去调用server里面的方法。虽然是有通用mapper方法，但是每次添加一个server方法之后都要添加对应的mapper方法，这样开发的也显得比较繁琐，所以我们需要一个通用server类，用这个类去调用第二步的方法就可以了。
```
public interface BaseService<T> {
	/**
	 * 查询所有
	 * 
	 * @return 返回所有数据
	 */
	List<T> findAll();

	/**
	 * 添加
	 * 
	 * @param t   实体
	 *          
	 * @return
	 */
	int save(T t);

	/**
	 * 修改
	 * 
	 * @param t
	 *            实体
	 * @return
	 */
	int updateByPrimaryKey(T t);

	/**
	 * 根据主键删除
	 * 
	 * @param t   主键
	 *            
	 * @return
	 */
	int deleteByPrimaryKey(int t);
	
	/**
	 * 查询表格列表
	 * @param t 分页参数
	 * @return
	 */
	TableData<T> getTableData(PageBean pageBean);
}
```
上面只是封装基本增删改查的方法，后续可自行添加方法。
    然后添加实现类
```
public abstract class BaseServiceImpl<T> implements BaseService<T> {
	@Autowired
	protected MyMapper<T> mapper;

	@Override
	public List<T> findAll() {
		return mapper.selectAll();
	}

	@Override
	public int save(T t) {
		return mapper.insert(t);
	}

	@Override
	public int updateByPrimaryKey(T t) {
		return mapper.updateByPrimaryKey(t);
	}

	@Override
	public int deleteByPrimaryKey(int t) {
		return mapper.deleteByPrimaryKey(t);
	}

	@Override
	public TableData<T> getTableData(PageBean bean) {
		int count = mapper.selectAll().size();
		if (count > 0) {
			PageHelper.startPage((bean.getOffset()/bean.getLimit()) + 1, bean.getLimit());
			List<T> list = this.findAll();
			return TableData.bulid(count, list);
		}

		return TableData.empty();
	}
}

```
注意：我用的编辑器是eclipse，如果用idea编辑器，这里可把abstract去掉。

然后添加对应的接口和实现类继承上面的接口和方法就可以了，比如添加一个newsserver 接口和newsserverImpl类
```
public interface NewsService extends BaseService<SysUser> {

}
```
```
@Service
public class NewsServiceImpl extends BaseServiceImpl<SysUser> implements NewsService{

}
```
5 为了减少数据库服务器的压力，一般我们查询数据的时候都会使用pagehelper进行分页查询，为了更加清晰的显示我们展示的数据，使用bootstrap table展示数据，bootstrap table获取数据有两种途经，一种是客户端模式，即获取全部数据之后，在前端进行分页展示。另外一种，也就是我们接下来要说的服务端模式：要获取的数据信息，比如获取数据页码，每一页数据的大小，都可以通过前端发送以上的参数向后台发请求，后台得到这些参数信息之后返回数据。
6 引入bootstrap table相关的js css文件之后，开始在网上找了一些资料之后发现很多都是要在前端页面添加如下繁琐的配置，
```
       $('#mytable').bootstrapTable({
                 //请求方法
                method: 'get',
                 //是否显示行间隔色
                striped: true,
                //是否使用缓存，默认为true，所以一般情况下需要设置一下这个属性（*）     
                cache: false,    
                //是否显示分页（*）  
                pagination: true,   
                 //是否启用排序  
                sortable: false,    
                 //排序方式 
                sortOrder: "desc",    
                //初始化加载第一页，默认第一页
                //我设置了这一项，但是貌似没起作用，而且我这默认是0,- -
                //pageNumber:1,   
                //每页的记录行数（*）   
                pageSize: 10,  
                //可供选择的每页的行数（*）    
                pageList: [10, 25, 50, 100],
                //这个接口需要处理bootstrap table传递的固定参数,并返回特定格式的json数据  
                url: "${contextPath}/mapper/getTableData",
                //默认值为 'limit',传给服务端的参数为：limit, offset, search, sort, order Else
                //queryParamsType:'',   
                ////查询参数,每次调用是会带上这个参数，可自定义                         
                queryParams: queryParams : function(params) {
                    var subcompany = $('#subcompany option:selected').val();
                    var name = $('#name').val();
                    return {
                          pageNumber: params.offset+1,
                          pageSize: params.limit,
                          companyId:subcompany,
                          name:name
                        };
                },
                //分页方式：client客户端分页，server服务端分页（*）
                sidePagination: "server",
                //是否显示搜索
                search: false,  
                //Enable the strict search.    
                strictSearch: true,
                //Indicate which field is an identity field.
                idField : "id",
                columns: [],
                pagination:true
            });
```
每次添加一个页面如果都要添加以上的配置信息也显得繁琐，不过bootstrap-table.js里面有个默认的配置，只需要修改里面的几个配置。
```
 contentType: 'application/json',//post请求头 application/x-www-form-urlencoded; charset=UTF-8'
 dataType: 'json',
 sidePagination: 'server', // 改成server       
```
当我们点击表格分页页码的时候，获取改变每页显示的页码时候，前端会自动调用queryParams()方法，我们需要将这些数据传递给后台，
```
       function queryParams(params) {
			var query={};
			query["limit"] = params.limit;//第几条数据开始
			query["offset"] = params.offset;//数据大小
			return query;
		}
```
6 配合上一步前端的分页，我们就需要使用pagehelp插件了，同样我们把这个分页的方法放在通用server类上，
```
 public TableData<T> getTableData(PageBean bean) {
        int count = mapper.selectAll().size();
        if (count > 0) {
            PageHelper.startPage((bean.getOffset()/bean.getLimit()) + 1, bean.getLimit());
            List<T> list = this.findAll();
            return TableData.bulid(count, list);
        }

        return TableData.empty();
    }
```
上面的pagehelper.startpage需要做一点改变，前端传过来的是显示第几条数据，但是startpage方法第一个参数是显示第几页的数据，所以做一个转换pageoffset/limit  +1，然后在查询数据，需要注意的是，一定要将startpage方法方法查询数据语句的前一行，不能空行，或者换行。
###附录:

[github源码](https://github.com/jeremylai7/springboot-bootstrap.git)

[demo展示](https://www.jeremy7.cn/bootstrap/)

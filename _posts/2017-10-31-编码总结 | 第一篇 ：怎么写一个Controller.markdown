---
layout:     post
title:      "编码总结 | 第一篇"
subtitle:   "怎么写一个好的Controller"
date:       2017-10-31
author:     "BENJAMIN"
header-img: ""
tags:
    - 编码规范
    
---

```
    业务发开中，其实大部分的代码用的技术都不是很深，大部分呢都是基本的CRUD的开发，但是很矛
盾的是为什么大家还是苦逼的加班，熬夜。一方面工作不复杂，一方面却累成狗，问题到底出在哪里，时间
都花到哪里去了？
	其实大部分人的大部分时间都是在 定位问题 + 改代码，真正开发的时间并不多。定位问题包括开发转
测试的时候发现问题和上线后发现问题，改代码的包括改bug和因为需求变动修改代码。
	其实，对于个人来说，技术很重要，但是对于工作来说，编码的习惯比技术更加主要。工作中你面试的
大部分技术都不需要用到的。工作中，因为你的编码习惯不好，写的代码质量差，代码冗余重复多，很多无
关的代码和业务代码搅在一起，导致了你疲于奔命应付各种问题。
	后面我会把我们系统中大家编码的问题一个一个写出来，并把我的解决办法分享出来。
```


##第一篇 关于Controller
Controller也就是API接口，工作中少不了要定义接口，系统间的、前后台的。
但是经常会看到很多不规范的接口定义：  

### 返回的格式不统一

```
	同一个接口，有时候返回数组，有时候返回单个；成功的时候返回对象，失败的时候返回错误信息字符
串。工作中有个系统集成就是这样定义的接口，真是辣眼睛。这个对应代码上，返回的类型是map，json，
object，都是不应该的。实际工作中，应该定义一个统一的格式，就是ResultBean
```
错误范例：

```
 //返回map可读性不好，尽量不要
　＠PostMapping("/delete")
  public Map<String, Object> delete(long id, String lang) {

  }

  // 成功返回boolean，失败返回string，大忌
  ＠PostMapping("/delete")
  public Object delete(long id, String lang) {
    try {
      boolean result = configService.delete(id, local);
      return result;
    } catch (Exception e) {
      log.error(e);
      return e.toString();
    }
  }
```

### 没有考虑失败情况

一开始只考虑成功场景，等后面测试发现有错误情况，怎么办，改接口呗，前后台都改，劳民伤财无用功。

错误范例：

```
 //不返回任何数据，没有考虑失败场景，容易返工
　＠PostMapping("/update")
  public void update(long id, xxx) {

  }
```

### 出现和业务无关的输入参数

当前用户信息不应该出现参数里面，应该从当前会话里面获取。后面讲ThreadLocal会说到怎么样去掉。除了代码可读性不好问题外，尤其是参数出现当前用户信息的，这是个严重问题。
错误范例：

```
  //（当前用户删除数据）参数出现lang和userid，尤其是userid，大忌
　＠PostMapping("/delete")
  public Map<String, Object> delete(long id, String lang, String userId) {

  }
```

### 出现复杂的输入参数

一般情况下，不允许出现例如json字符串这样的参数，这种参数可读性极差。应该定义对应的bean。

错误范例：

```
 // 参数出现json格式，可读性不好，代码也难看
　＠PostMapping("/update")
  public Map<String, Object> update(long id, String jsonStr) {

  }
```

### 没有返回应该返回的数据

例如，新增接口一般情况下应该返回新对象的id标识。别人要不要是别人的事情，你该返回的还是应该返回。

错误范例：

```
 // 约定俗成，新建应该返回新对象的信息，只返回boolean容易导致返工
　＠PostMapping("/add")
  public boolean add(xxx) {
    //xxx
    return configService.add();
  }
```

## 如何解决呢

定义一个通用的返回bean：

```
/**
 * 接口通用的返回bean
 * 
 * @author BENJAMIN
 *
 * @param <T>
 */
public class ResultBean<T> implements Serializable {

	/**
	 *
	 * @author Administrator TODO description
	 * @date 2017年4月27日
	 */

	public ResultBean() {
		super();
	}

	/**
	 * 
	 */
	private static final long serialVersionUID = -1092646848721671618L;

	/**
	 * 通用API调用成功
	 */
	public final static Integer success = 200;

	/**
	 * 通用的验证参数失败
	 */
	public final static Integer validFaild = 400;

	private long ElapsedMilliseconds;

	public long getElapsedMilliseconds() {
		return ElapsedMilliseconds;
	}

	public void setElapsedMilliseconds(long elapsedMilliseconds) {
		ElapsedMilliseconds = elapsedMilliseconds;
	}

	public final static Integer errorUnknown = 900;

	public final static Integer errorDB = 901;

	private Integer resultCode;

	public Integer getResultCode() {
		return resultCode;
	}

	public void setResultCode(Integer resultCode) {
		this.resultCode = resultCode;
	}

	public T getData() {
		return data;
	}

	public void setData(T data) {
		this.data = data;
	}

	public String getErrMsg() {
		return errMsg;
	}

	public void setErrMsg(String errMsg) {
		this.errMsg = errMsg;
	}

	private T data;

	private String errMsg;

	/**
	 * 失败
	 * 
	 * @param resultCode
	 * @param errMsg
	 */
	public ResultBean(Integer resultCode, String errMsg) {
		super();
		// 创建日期对象
		Date d = new Date();
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		String date = sdf.format(d);
		this.resultCode = resultCode;
		this.errMsg = errMsg;
		this.isSuccess = false;
		this.time = date;
	}

	/**
	 * 成功
	 * 
	 * @param resultCode
	 * @param data
	 * @param errMsg
	 */
	public ResultBean(Integer resultCode, T data, String errMsg) {
		super();
		// 创建日期对象
		Date d = new Date();
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		String date = sdf.format(d);
		this.resultCode = resultCode;
		this.data = data;
		this.errMsg = errMsg;
		this.isSuccess = true;
		this.time = date;
	}

	private boolean isSuccess;

	private String time;

	/**
	 * @return the time
	 */
	public String getTime() {
		return time;
	}

	/**
	 * @param time
	 *            the time to set
	 */
	public void setTime(String time) {
		this.time = time;
	}

	public boolean isSuccess() {
		return isSuccess;
	}

	public void setSuccess(boolean isSuccess) {
		this.isSuccess = isSuccess;
	}
}
```

通过一个AOP来环绕所有的API

```
@Component
@Aspect
@Order(5)
public class ApiAspect {
	private Logger logger = LoggerFactory.getLogger(ApiAspect.class);

	@Around("@annotation(com.yonyou.cloud.common.annotation.YcApi)")
	public Object logServiceAccess(ProceedingJoinPoint pjp) throws Throwable {
		long start = System.currentTimeMillis();

		String className = pjp.getTarget().getClass().getName();
		String methodName = pjp.getSignature().getName();
		String fullMethodName = className + "." + methodName;
		boolean needLog = false;
		
		logger.info(fullMethodName + "将被调用");

		Object result = null;
		try {
			result = pjp.proceed();
			if (result instanceof ResultBean<?>) {
				((ResultBean<?>) result).setSuccess(true);
			}
		} catch (Throwable e) {

			if (result != null && result instanceof ResultBean) {
				ResultBean<?> errorResult = (ResultBean<?>) result;
				if (NumberUtil.isBlankChar(errorResult.getResultCode())) {
					errorResult.setResultCode(ResultBean.errorUnknown);
				}
				if (StrUtil.isEmpty(errorResult.getErrMsg())) {
					errorResult.setErrMsg(e.getLocalizedMessage());
				}
			} else {
				if (e instanceof BizException) {
					result = new ResultBean<Object>(((BizException) e).getCode(), e.getMessage());
				} else {
					result = new ResultBean<Object>(ResultBean.errorUnknown, e.getMessage());
				}
			}
			logger.error(fullMethodName + "执行出错,详情:", e);
		}

		long end = System.currentTimeMillis();
		long elapsedMilliseconds = end - start;
		if (needLog) {
			logger.info(fullMethodName + "执行耗时:" + elapsedMilliseconds + " 毫秒");
		}

		if (result != null && result instanceof ResultBean) {
			((ResultBean<?>) result).setElapsedMilliseconds(elapsedMilliseconds);
		}
		try {
			if (result != null && result instanceof ResultBean) {
				ResultBean<?> e = (ResultBean<?>) result;
				Object data = e.getData();
				logger.info("返回值：" + data.toString());
			}

		} catch (Exception e) {
			logger.error("error", e);
		}
		return result;
	}
```

这样我们的Controller 就简化成:

```
@GetMapping("/all")
	public ResultBean<Collection<Config>> getAll() {
		return new ResultBean<Collection<Config>>(configService.getAll());
	}

	@PostMapping("/add")
	public ResultBean<Long> add(Config config) {
		return new ResultBean<Long>(configService.add(config));
	}

	@PostMapping("/delete")
	public ResultBean<Boolean> delete(long id) {
		return new ResultBean<Boolean>(configService.delete(id));
	}
	
```

## 总结
Controller中的总结：  

1 所有函数返回统一的ResultBean格式

没有统一格式，AOP无法玩。

2 ResultBean是controller专用的，不允许往后传！

3 Controller做参数格式的转换，不允许把json，map这类对象传到services去，也不允许services返回json、map。

一般情况下！写过代码都知道，map，json这种格式灵活，但是可读性差，如果放业务数据，每次阅读起来都比较困难。定义一个bean看着工作量多了，但代码清晰多了。


4 参数中一般情况不允许出现Request，Response这些对象

主要是可读性问题。一般情况下。

5 不需要打印日志

日志在AOP里面会打印，而且我的建议是大部分日志在Services这层打印。


下一篇我们来说说如何打印日志


关于规范的代码类库会放在这个git库中：
[https://github.com/ambluse/common-elegance](https://github.com/ambluse/common-elegance)



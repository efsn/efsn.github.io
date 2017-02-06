---
layout: post
title: "Freemarker"
category: NOTE
tags: [Freemarker]
---
* TOC
{:toc}
# Overview
> Configuration存储应用级设置，创建和缓存预解析Template

## Important
> cfg.setDirectory
> 在内部，变量封装为TemplateModel（object wrapping对象包装）

#### TemplateModel
> 4种类型的标量（boolean，number，string，date）

##### TemplateDateModel
> DATE TIME DATETIME UNKNOWN
> 当unkown，可以使用内置函数date、time、datetime，?string("yyyy-MM-dd")
> .now表示new Date()

##### TemplateDirectiveModel
> 实现自定义指令，Version2.3.11后加入代替TemplateTransformModel
>可以将常用的Directive作为共享变量存入Configuration中，也可以使用内置函数new

```ftl
<#assign upper="com.codeyn.UpperDirective"?new()>
```

**For example**
```java
public class UpperDirective implements TemplateDirectiveModel{
	public void execute(Environment env, Map params, TemplateModel[] loops, TemplateDirectiveBody body){
		if(!params.isEmpty()){
			throw new TemplateDirectiveException("Directive doesn't allow params");
		}
		if(loops.length > 0){
			throw new TemplateDirectiveException("Directive doesn't allow loop variables");
		}
		if(body != null){
			body.render(new UpperCaseFilterWriter(env.getOut()));
		}else{
			throw new RumtimeException("missing body");
		}
	}

	private static class UpperCaseFilterWriter extends Writer{
		private final Writer out;
		UpperCaseFilterWriter(Writer out){
			this.out = out;
		}
		public void write(char[] cbuf, int off, int len){
			char[] tcbuf = new char[len];
			for(int i = 0; i < len; i++){
				tcbuf[i] = Character.toUpperCase(cbuf[i+off]);
			}
			out.write(tcbuf);
		}
	}
}
```

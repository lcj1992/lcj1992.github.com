---
layout: post
title: url pattern
categories: java_web
tags: url_pattern
---

*   [标准的web.xml](#web.xml)
    *   [url-pattern种类](#kind)
    
*   [ant-style](#ant_style)

    
    

<h2 id="web.xml">标准的web.xml的path pattern</h2>

<h3 id="kind">url-pattern种类</h3>

|url-pattern|翻译下下|对应wrapper(匹配规则)|
|-|-|
|path.endsWith("/\*")*|以 /* 结尾的|`Wildcard wrapper`　通配符,匹配/xx /yy.xx|
|path.startsWith("\*.")|以 \*. 开头的|`Extension wrapper` 扩展,匹配xx.yy|
|path.equals("/")|/|`Default wrapper`|
|else | 以上3种之外的|`Exact wrapper` 精确匹配|

tomcat读取web.xml配置，调用addServletMappings将servlet加入`StandardContext.ServletMappings`中，tomcat自己有两个servlet，我们的web应用可以重写（web应用内url-pattern和servlet只能是一一对应的）

         <!-- The mapping for the default servlet -->
            <servlet-mapping>
                <servlet-name>default</servlet-name>
                <url-pattern>/</url-pattern>
            </servlet-mapping>

            <!-- The mappings for the JSP servlet -->
            <servlet-mapping>
                <servlet-name>jsp</servlet-name>
                <url-pattern>*.jsp</url-pattern>
                <url-pattern>*.jspx</url-pattern>
            </servlet-mapping>

处理完后结果如下图：其中key是url-pattern value是servlet-name

![url-pattern](/images/java_web/url_pattern.png)

addServletMapping会调用[addWrapper](#addWrapper)将对应模式url-pattern加入对应的`Mapper.ContextVersion.*wrappers`中``

![wrappers](/images/java_web/wrappers.png)

wrapper代表一个servlet，它负责管理一个Servlet，包括Servlet的装载,初始化,执行以及资源回收

<h3 id="handler">tomcat处理顺序</h3>

1.   精确匹配 `exact match`　对应 exactWrappers
2.   前缀匹配 `prefix match`　对应wildcardWrappers
3.   扩展匹配 `extension　match` 对应extensionWrappers
4.   使用资源文件来处理servlet，使用contextVersion的welcomeResources属性，这个属性是个字符串数组
    *   精确匹配
    *   前缀匹配
    *   文件夹
7.   默认的servlet来处理　对应default wrapper

源码详见[internalMapWrapper](#internalMapWrapper)

## ant_style

Ant path 匹配原则
路径匹配原则(Path Matching) Spring MVC中的路径匹配要比标准的web.xml要灵活的多。默认的策略实现了 org.springframework.util.AntPathMatcher，就像名字提示的那样，路径模式是使用了Apache Ant的样式路径，Apache Ant样式的路径有三种通配符匹配方法（在下面的表格中列出)
这些可以组合出很多种灵活的路径模式

|Wildcard	|Description	 |
|-|-|
|?	|匹配任何单字符	 |
|*	|匹配0或者任意数量的字符	 |
|**	|匹配0或者更多的目录	 |

例子

|Path	|Description	 |
|-|-|
|/app/*.x	|匹配(Matches)所有在app路径下的.x文件	 |
|/app/p?ttern	|匹配(Matches) /app/pattern 和 /app/pXttern,但是不包括/app/pttern
|/**/example	|匹配(Matches) /app/example, /app/foo/example, 和 /example
|/app/**/dir/file.	| 匹配(Matches) /app/dir/file.jsp, /app/foo/dir/file.html,/app/foo/bar/dir/file.pdf, 和 /app/dir/file.java
|/**/*.jsp	|　匹配(Matches)任何的.jsp 文件

<h2 id="addition">附录</h2>

addWrapper：tomcat读取web.xml，将servlet分别加入对应的wrapper中
internalMapWrapper: 按照一定的处理顺序，将请求分别打给对应的wrapper，交给该wrapper的servlet处理

<h3 id="addWrapper">addWrapper</h3>

        /**
         * Adds a wrapper to the given context.
         *
         * @param context The context to which to add the wrapper
         * @param path Wrapper mapping
         * @param wrapper The Wrapper object
         * @param jspWildCard true if the wrapper corresponds to the JspServlet
         * @param resourceOnly true if this wrapper always expects a physical
         *                     resource to be present (such as a JSP)
         * and the mapping path contains a wildcard; false otherwise
         */
        protected void addWrapper(ContextVersion context, String path,
                Object wrapper, boolean jspWildCard, boolean resourceOnly) {
            synchronized (context) {
                Wrapper newWrapper = new Wrapper();
                newWrapper.object = wrapper;
                newWrapper.jspWildCard = jspWildCard;
                newWrapper.resourceOnly = resourceOnly;
                if (path.endsWith("/*")) {
                    // Wildcard wrapper
                    newWrapper.name = path.substring(0, path.length() - 2);
                    Wrapper[] oldWrappers = context.wildcardWrappers;
                    Wrapper[] newWrappers =
                        new Wrapper[oldWrappers.length + 1];
                    if (insertMap(oldWrappers, newWrappers, newWrapper)) {
                        context.wildcardWrappers = newWrappers;
                        int slashCount = slashCount(newWrapper.name);
                        if (slashCount > context.nesting) {
                            context.nesting = slashCount;
                        }
                    }
                } else if (path.startsWith("*.")) {
                    // Extension wrapper
                    newWrapper.name = path.substring(2);
                    Wrapper[] oldWrappers = context.extensionWrappers;
                    Wrapper[] newWrappers =
                        new Wrapper[oldWrappers.length + 1];
                    if (insertMap(oldWrappers, newWrappers, newWrapper)) {
                        context.extensionWrappers = newWrappers;
                    }
                } else if (path.equals("/")) {
                    // Default wrapper
                    newWrapper.name = "";
                    context.defaultWrapper = newWrapper;
                } else {
                    // Exact wrapper
                    if (path.length() == 0) {
                        // Special case for the Context Root mapping which is
                        // treated as an exact match
                        newWrapper.name = "/";
                    } else {
                        newWrapper.name = path;
                    }
                    Wrapper[] oldWrappers = context.exactWrappers;
                    Wrapper[] newWrappers =
                        new Wrapper[oldWrappers.length + 1];
                    if (insertMap(oldWrappers, newWrappers, newWrapper)) {
                        context.exactWrappers = newWrappers;
                    }
                }
            }
        }


<h3 id="internalMapWrapper">internalMapWrapper</h3>

        /**
         * Wrapper mapping.
         */
        private final void internalMapWrapper(ContextVersion contextVersion,
                                              CharChunk path,
                                              MappingData mappingData)
            throws Exception {

            int pathOffset = path.getOffset();
            int pathEnd = path.getEnd();
            int servletPath = pathOffset;
            boolean noServletPath = false;

            int length = contextVersion.path.length();
            if (length != (pathEnd - pathOffset)) {
                servletPath = pathOffset + length;
            } else {
                noServletPath = true;
                path.append('/');
                pathOffset = path.getOffset();
                pathEnd = path.getEnd();
                servletPath = pathOffset+length;
            }

            path.setOffset(servletPath);

            // Rule 1 -- Exact Match
            Wrapper[] exactWrappers = contextVersion.exactWrappers;
            internalMapExactWrapper(exactWrappers, path, mappingData);

            // Rule 2 -- Prefix Match
            boolean checkJspWelcomeFiles = false;
            Wrapper[] wildcardWrappers = contextVersion.wildcardWrappers;
            if (mappingData.wrapper == null) {
                internalMapWildcardWrapper(wildcardWrappers, contextVersion.nesting,
                                           path, mappingData);
                if (mappingData.wrapper != null && mappingData.jspWildCard) {
                    char[] buf = path.getBuffer();
                    if (buf[pathEnd - 1] == '/') {
                        /*
                         * Path ending in '/' was mapped to JSP servlet based on
                         * wildcard match (e.g., as specified in url-pattern of a
                         * jsp-property-group.
                         * Force the context's welcome files, which are interpreted
                         * as JSP files (since they match the url-pattern), to be
                         * considered. See Bugzilla 27664.
                         */
                        mappingData.wrapper = null;
                        checkJspWelcomeFiles = true;
                    } else {
                        // See Bugzilla 27704
                        mappingData.wrapperPath.setChars(buf, path.getStart(),
                                                         path.getLength());
                        mappingData.pathInfo.recycle();
                    }
                }
            }

            if(mappingData.wrapper == null && noServletPath) {
                // The path is empty, redirect to "/"
                mappingData.redirectPath.setChars
                    (path.getBuffer(), pathOffset, pathEnd-pathOffset);
                path.setEnd(pathEnd - 1);
                return;
            }

            // Rule 3 -- Extension Match
            Wrapper[] extensionWrappers = contextVersion.extensionWrappers;
            if (mappingData.wrapper == null && !checkJspWelcomeFiles) {
                internalMapExtensionWrapper(extensionWrappers, path, mappingData,
                        true);
            }

            // Rule 4 -- Welcome resources processing for servlets
            if (mappingData.wrapper == null) {
                boolean checkWelcomeFiles = checkJspWelcomeFiles;
                if (!checkWelcomeFiles) {
                    char[] buf = path.getBuffer();
                    checkWelcomeFiles = (buf[pathEnd - 1] == '/');
                }
                if (checkWelcomeFiles) {
                    for (int i = 0; (i < contextVersion.welcomeResources.length)
                             && (mappingData.wrapper == null); i++) {
                        path.setOffset(pathOffset);
                        path.setEnd(pathEnd);
                        path.append(contextVersion.welcomeResources[i], 0,
                                contextVersion.welcomeResources[i].length());
                        path.setOffset(servletPath);

                        // Rule 4a -- Welcome resources processing for exact macth
                        internalMapExactWrapper(exactWrappers, path, mappingData);

                        // Rule 4b -- Welcome resources processing for prefix match
                        if (mappingData.wrapper == null) {
                            internalMapWildcardWrapper
                                (wildcardWrappers, contextVersion.nesting,
                                 path, mappingData);
                        }

                        // Rule 4c -- Welcome resources processing
                        //            for physical folder
                        if (mappingData.wrapper == null
                            && contextVersion.resources != null) {
                            Object file = null;
                            String pathStr = path.toString();
                            try {
                                file = contextVersion.resources.lookup(pathStr);
                            } catch(NamingException nex) {
                                // Swallow not found, since this is normal
                            }
                            if (file != null && !(file instanceof DirContext) ) {
                                internalMapExtensionWrapper(extensionWrappers, path,
                                                            mappingData, true);
                                if (mappingData.wrapper == null
                                    && contextVersion.defaultWrapper != null) {
                                    mappingData.wrapper =
                                        contextVersion.defaultWrapper.object;
                                    mappingData.requestPath.setChars
                                        (path.getBuffer(), path.getStart(),
                                         path.getLength());
                                    mappingData.wrapperPath.setChars
                                        (path.getBuffer(), path.getStart(),
                                         path.getLength());
                                    mappingData.requestPath.setString(pathStr);
                                    mappingData.wrapperPath.setString(pathStr);
                                }
                            }
                        }
                    }

                    path.setOffset(servletPath);
                    path.setEnd(pathEnd);
                }

            }

            /* welcome file processing - take 2
             * Now that we have looked for welcome files with a physical
             * backing, now look for an extension mapping listed
             * but may not have a physical backing to it. This is for
             * the case of index.jsf, index.do, etc.
             * A watered down version of rule 4
             */
            if (mappingData.wrapper == null) {
                boolean checkWelcomeFiles = checkJspWelcomeFiles;
                if (!checkWelcomeFiles) {
                    char[] buf = path.getBuffer();
                    checkWelcomeFiles = (buf[pathEnd - 1] == '/');
                }
                if (checkWelcomeFiles) {
                    for (int i = 0; (i < contextVersion.welcomeResources.length)
                             && (mappingData.wrapper == null); i++) {
                        path.setOffset(pathOffset);
                        path.setEnd(pathEnd);
                        path.append(contextVersion.welcomeResources[i], 0,
                                    contextVersion.welcomeResources[i].length());
                        path.setOffset(servletPath);
                        internalMapExtensionWrapper(extensionWrappers, path,
                                                    mappingData, false);
                    }

                    path.setOffset(servletPath);
                    path.setEnd(pathEnd);
                }
            }


            // Rule 7 -- Default servlet
            if (mappingData.wrapper == null && !checkJspWelcomeFiles) {
                if (contextVersion.defaultWrapper != null) {
                    mappingData.wrapper = contextVersion.defaultWrapper.object;
                    mappingData.requestPath.setChars
                        (path.getBuffer(), path.getStart(), path.getLength());
                    mappingData.wrapperPath.setChars
                        (path.getBuffer(), path.getStart(), path.getLength());
                }
                // Redirection to a folder
                char[] buf = path.getBuffer();
                if (contextVersion.resources != null && buf[pathEnd -1 ] != '/') {
                    Object file = null;
                    String pathStr = path.toString();
                    try {
                        file = contextVersion.resources.lookup(pathStr);
                    } catch(NamingException nex) {
                        // Swallow, since someone else handles the 404
                    }
                    if (file != null && file instanceof DirContext) {
                        // Note: this mutates the path: do not do any processing
                        // after this (since we set the redirectPath, there
                        // shouldn't be any)
                        path.setOffset(pathOffset);
                        path.append('/');
                        mappingData.redirectPath.setChars
                            (path.getBuffer(), path.getStart(), path.getLength());
                    } else {
                        mappingData.requestPath.setString(pathStr);
                        mappingData.wrapperPath.setString(pathStr);
                    }
                }
            }

            path.setOffset(pathOffset);
            path.setEnd(pathEnd);

        }

[1]<http://www.cnblogs.com/fangjian0423/p/servletcontainer-tomcat-urlpattern.html>

[2]<http://stackoverflow.com/questions/4140448/difference-between-and-in-servlet-mapping-url-pattern>

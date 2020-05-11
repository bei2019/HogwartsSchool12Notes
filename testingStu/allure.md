# 安装
 - 官网地址：
    - https://github.com/allure-framework/allure-python
 - zip压缩包地址：
    - https://repo.maven.apache.org/maven2/io/qameta/allure/allure-commandline/2.13.3/
 - 解压安装下载下来的zip包，可见bin,config,lib,plugins四个目录
    - bin 目录的绝对路径放入path环境变量中
 - 验证安装是否成功
    - cmd窗口下 ，allure --version 返回的是安装的版本
    
# 依赖
- allure-pytest
> 不安装运行会报 pytest: error: unrecognized arguments: --alluredir=./resule/5

# 运行
- pytest --alluredir=/tmp/my_allure_results：
    - my_allure_results 是文件夹
    - 每条case生成一个json文件记录中执行结果
- pytest serve /tmp/my_allure_results
    - 根据生成的json文件生成html报告 并且启动默认浏览器
- allure generate ./result/ -o ./report/ -clean
    - ./result ： pytest执行后的json目录
    - -o ： 参数
    - ./report ：根据json生成html存储目录
    - clean: 覆盖路径加clean
- allure open -h 127.0.0.1 -p 8803 ./report/
    - 打开./report/中的测试报告
    - 类似于启动了一个tomcat服务
    
# allure 常用特性
## 标注功能
- @Feature
    - 功能上加@allure.feature("功能名称")
- @story
    - 子功能上加上@allure.story("子功能名称")
- @step
    - 步骤上加@allure.step("步骤细节")
- @attach
    - @allure.attach("具体文本信息")
- 如果只测试登录功能，运行时可加限制过滤
    - pytest 文件名 --allure_features '功能名称' --allure_stories'子功能名称'
    
## allure feature/story/step
- feature相当于一个功能，一个大的模块，将case分类到莫格feature中，报告中behaviore中显示，相当于testsuite
- story相当于对应这个功能或者模块下的不同场景，分支功能，属于feature之下的结构，报告在features中显示，相当于testcase
- feature与story类似于父子关系   
- step：
    - 测试过程中每个步骤一般放在具体逻辑中，可以放在关键步骤中，在报告中显示
    - @allure.step()只能以装饰器的形式放在类或者方法上面
    - with allure.step():可以放在测试用例里面，但测试步骤的代码需要被该语句包含
    
 
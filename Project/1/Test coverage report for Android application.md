#Android应用生成测试覆盖率报告

对于使用AndroidJUnit4 runner创建的Android集成测试用例，之前一直都没有发现，一种合适的产生代码覆盖率的方式。我曾经尝试过很多方式，但是要不就是不奏效，要不就是只合适我现在已经不再使用的Robolectric测试框架，其他开发者，也渐渐不再新项目中使用这个测试框架了。比如Square的Sqlbrite项目，现在已经开始采用AndroidJUnit4 runner进行他们的项目测试。最近，我在Reddit上面发现了一个有趣的讨论，从中我发现了一个不需要额外插件，脚本和多行配置就可以简单生成代码覆盖率的方式，Android SDK 现在已经内置了对 [Emma Test Coverage](http://emma.sourceforge.net/)框架的支持，可以在官方文档中进行查阅。

我们只要做的一件事，就是在 `build.gradle` 中应用`jacoco—android`插件。

```
apply plugin: 'jacoco-android'
```
然后参照如下示例，将 `testCoverageEnabled` 设置为 `true`

```
android {
   buildTypes {
      debug {
         testCoverageEnabled = true
      }
   }
}
```
为了能生成代码覆盖率报告，我们需要将Android设备或者模拟器连接到计算机，因为 在生成报告前，会执行 `connectedCheck` 任务。

之后，我们可以执行如下的命令行

```
./gradlew createDebugCoverageReport
```
该任务会分析 `/src/main/java/` 路径下的代码和 `/src/androidTest/java/` 目录下单元测试用例。

在执行这个任务之后，我们可以在模块的如下路径中找到代码覆盖率报告

```
/build/outputs/reports/coverage/debug/
```
我们可以在浏览器中打开 index.html 文件，可以看见可视化的报告。
同时，在同一级目录下，我们也可以找到可以供持续集成覆盖率分析使用的 report.xml 文件。

除了上面提到的文件，Gradle也会在如下的路径创建 `coverage.ec` 文件。

```
/build/outputs/code-coverage/connected/
```

某些情况下，我们可能需要这些文件，比如在选择Jenkins插件或者别的工具生成代码覆盖率的时候。

下面，你能看见一个 Android 开源项目，[Presfer](https://github.com/pwittchen/prefser)，的代码覆盖率报告。

![](http://blog.wittchen.biz.pl/wp-content/uploads/2015/06/prefser_test_coverage_report_03.06.2015.png)

这是一份由 [JaCoCo](http://www.eclemma.org/jacoco/) 代码覆盖率库生成的报告。

通过分析报告，我增加了部分新的测试用例，小幅度优化了部分代码，使得项目的覆盖率达到了 100%。

![](http://blog.wittchen.biz.pl/wp-content/uploads/2015/06/prefser_test_coverage_04.06.2015.png)

为了能在 Jenkins CI 上发布报告，可以使用代码覆盖率插件，但是并不能确认插件的稳定性。另一种解决方案是  [HTML Publisher plugin](https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin)，我们可以增加相应动作，在 Jenkins 的任务中，通过默认的HTML界面产生覆盖率报告，我认为这是一种非常方便的方式，易于创建，并且方便进行代码导航，能定位到没有覆盖的代码行，方法和分支。

我们可以方便的观察到Android项目的代码覆盖情况通过这种简洁快捷的方式，能帮助我们最终发现项目的瓶颈所在，增加应用或者lib库的整体质量
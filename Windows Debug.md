Windows Debug 

**VS**使用**administrator**启动调试，不然有些**P/Invoke**调用会失败

- 下载调试环境资源

  资源包路径：http://10.12.23.54/view/Nightly_x64/job/Receiver-2019_x64/

  note：目前资源包拉取的webr是http://10.12.20.54/jenkins/job/WebR-Vue/， 不是nightly版本,如果需要可以修改

- 解压资源包到`trunk\prj\NewTVUTransport\OutputDebug`
- 替换自己的Config.xml和libraryconfig.xml
- 修改`VU.App.ConsoleReceiver_v15`的debug profile， 选择Executable类型并设置启动调试程序的路径`trunk\prj\NewTVUTransport\OutputDebug\TVU.App.ConsoleReceiver.exe`
- 替换nginx配置文件，同时修改配置文件中的webr的路径
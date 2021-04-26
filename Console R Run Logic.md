# Console R Run Logic

- Main

  - Run

    1. read tvu service list and init  **TVUServiceListHelper.Init** ,update with ***libraryconfig.xml***

    2. read cloud service list and init **TVUCloudServiceHelper.Init**

    3. get the transceiver id by command line argument or libpeerid

    4. init raven with product version **InitRaven(productVersion.StrBuild, transceiverID)**

    5. app int **AppInit().Init(args)**

       - authorize the app(set the firewall) only **windows**

       - get system information **SystemConfigurationUtility.GetSystemConfigurationInfo()**

       - set current directory **Directory.SetCurrentDirectory(AppDomain.CurrentDomain.BaseDirectory)**

       - set connection limit count **SetConnectionLimit()**

       - create necessary directories **TVUDirectoryInfos.Init()**

       - kill parentless process  **KillProcesses**  

         ``` c#
         "switcher", "seamless_switching", "player", "playeroftvu", "vlc", "ffmpeg", "m3u8", "ext-dl", "R2TPDSAgent", "TVU.ThumbnailCreation.Creator3", "TVUFecServer", "dcap2", "FlvGoLive", "peerclient", "libcoreserver", "TVURouter", "Decoder"
         
         { "TVU264", "TVU264-{0}" }
         ```

       - set registry to do not show the error UI **WriteRegistry**,   only **windows**

    6. appcore load

       - set base data folder

       - create some folders if not exist, like *Conf7,EncodingProfile2,FTPUploadProfile2*

       - read the *Conf7/Recipe.tml*

       - start a new thread to continue load

         1. init SystemdCmdExecuteService   only **Linux**

         2. app start

            - generate the application session **GUID**

            - get the instance index, because we can install multiple transceivers  only **Linux**

            - create LibCoreControl instance with TransceiverID and index

              1. set urls
              2. create **PInvokeProxy** instance

            - load the **TVU.App.ReceiverProductVersion.dll** to get the version information

            - **LibCoreControl** init    **RX?.InitLibCore(productBuildVersion, DownloadDir)**

              1. generate libcore initiation configuration

                ```c#
                conf = new LibCoreInitConfiguration()
                {
                    ProductVersion = productBuildVersion,
                    // TODO: Yes we always use Windows.
                    ReceiverType = EnumReceiverType.Windows,
                    DownloadDirectory = downloadDir,
                    BaseDirectory = AppDomain.CurrentDomain.BaseDirectory,
                    UserID = $"{AppCore.Instance.TransceiverID}",
                    LibCoreDockerVersion = DependencyInfo.Instance.DockerLibCoreVersion,
                           
                    PeerServiceHost = AppCoreConfig.Instance.RPSIP,
                    PeerServicePort = AppCoreConfig.Instance.RPSPort,
                    ExternalIP = Entity.CoreRuntimeStatus.ModuleConfig.Instance.RExternalIP,
                    ExternalPort = PortMappings.Instance.Transport.ExternalPort,
                    LocalIP = Entity.CoreRuntimeStatus.ModuleConfig.Instance.RLocalIP,
                    LocalPort = PortMappings.Instance.Transport.LocalPort,
                    RUrl = OutputHTTPUrl,
                    RPlayShmUrl = OutputSHMUrl,
                
                    FakePeerID = ReceiverID,
                    AfterLibCoreStarted = ac,
                };
                ```

                

              2. **PInvokeProxy** init  **PInvokeProxy.Init(conf)**

                - generate the init action and combine the **after libcore started** in the init configuration

                - generate the mini tiny configuration

                   ```c#
                   new MiniTinyConfiguration(
                       conf.DownloadDirectory, //string downloadDirectory
                       conf.BaseDirectory, //string baseDirectory
                       conf.UserID, //string userID
                       ModuleConfig.Instance.EnableDocker, //bool enableDocker
                       conf.LibCoreDockerVersion, //string libCoreDockerVersion
                       0, //int index
                       conf.FakePeerID, //string fakePeerIDHex
                       "0", //string instanceIdentifier
                       AppDomain.CurrentDomain.BaseDirectory, //string workingDirectory
                       conf.PeerServiceHost, //string peerServiceHost
                       (ushort)conf.PeerServicePort, //ushort peerServicePort
                       conf.ExternalIP, //string externalIP
                       (ushort)conf.ExternalPort, //ushort externalPort
                       conf.LocalIP, //string localIP
                       (ushort)conf.LocalPort, //ushort localPort
                       ModuleConfig.Instance.PlayPort, //int playPort
                       ModuleConfig.Instance.FecTransportRpcServerPort, //ushort fecPort
                       ModuleConfig.Instance.FecTransportHttpProxyPort, //ushort fecHttpProxyPort
                       ModuleConfig.Instance.GrpcControlPort //ushort libCoreGrpcControlPort
                       //int libCoreGrpcAgentVersion = 0
                   );
                   ```

                - generate the **MiniTinyReceiver**

                   1. kill parentless process 

                     ```c#
                     TVUKillProcesses.DoParentlessKill(
                         new List<string>()
                         {
                             "libcoreserver",
                             "python", // For PygRPC.
                             "TVUFecServer",
                         });
                     ```

                   2. generate the libcore server by `config.LibCoreGrpcAgentVersion` 

                     ```c#
                     switch (config.LibCoreGrpcAgentVersion)
                     {
                         case 1:
                             PygRPC = new PygRPCServerStarter(
                                 Config.InstanceIdentifier, 
                                 Config.WorkingDirectory, 
                                 Config.LibCoreGrpcControlPort);
                             break;
                         case 0:
                         default:
                             LibCore2 = new LibCoreServerStarter2(
                                 config.BaseDirectory, 
                                 config.DownloadDirectory, 
                                 config.UserID, 
                                 config.LibCoreDockerVersion, 
                                 config.EnableDocker, 
                                 Config.InstanceIdentifier, 
                                 Config.WorkingDirectory, 
                                 Config.LibCoreGrpcControlPort);
                             break;
                     }
                     ```

                   3. generate FecServerStarter2

                     ```c#
                     Fec2 = new FecServerStarter2( 
                         Config.InstanceIdentifier, 
                         Config.WorkingDirectory, 
                         Config.FecPort, 
                         Config.LocalPort, 
                         Config.FecHttpProxyPort);
                     ```

                   4. generate LibCoreGrpcProxy | set the endpoint

                     ```c#
                     P1 = new LibCoreGrpcProxy($"127.0.0.1:{Config.LibCoreGrpcControlPort}");
                     ```

                - start **MiniTinyReceiver** subs `P1.StartSubs(true, false, initAction)`

                   1. generate the action

                   2. start libcore service `LibCore2?.Start(ac);`  ` "libcoreserver", $"tvu.libcoregrpc{instanceIdentifer}", workingDirectory`

                     - check if or not enable docker
                       1. start docker   **enable docker**
                       2. start process directly   **disable**
                         - start libcoreservice using Process   **windows**
                         - start libcoreservice by Systemd   **Linux**
                           1. add libcoreservice unit
                           2. reload Daemon

                   3. start libcore service `PygRPC?.Start(ac);`   `"EntryPoint2.py", $"tvu.pygrpc{instanceIdentifer}"`

                     - start process directly  

                       1. start libcoreservice using Process   **windows**
                       2. start libcoreservice by Systemd   **Linux**
                         - add libcoreservice unit
                         - reload Daemon

                       ```c#
                       if (SystemConfigurationInfo.IsWindows)
                       {
                           _taskHost = new GenericProcessInvokerQueue();
                           _taskHost.Invoke(
                               BinName, 
                               _args, 
                               1, 
                               false,        
                               Path.Combine(AppDomain.CurrentDomain.BaseDirectory, WorkingDirectory)
                           );
                           ac?.Invoke();
                       }
                       else
                       {
                           string serviceDesp = $"TVU Service for {BinName} {InstanceIdentifier}";
                           string newExecStart = $"{BinName} {_args}";
                           _systemdService = new SystemdFileItem(
                               InstanceIdentifier, 
                               serviceDesp, 
                               newExecStart, 
                               WorkingDirectory, 
                               null, 
                               KillMode, 
                               true, 
                               1);
                           _systemdService.AddUnitFile();
                           ServiceItem.ReloadDaemon();
                           _systemdService.Restart(ac);
                       }
                       ```

                       

                   4. start fec service `Fec2.Start();` `"TVUFecServer", $"tvu.fecserver{instanceIdentifer}", workingDirectory`

                     ```c#
                     if (withFec)
                         Fec2.Start();
                     ```

                   5. call the init action 

                     1. set peer id| send the command to libcore service |  `SetPeerID(fakeID)`
                     2. set product version | send the command to libcore service |  `SetProductVersion(conf.ProductVersion)`
                     3. set program type | send the command to libcore service | `SetProgramType(conf.ReceiverType)`
                     4. set download prefix | send the command to libcore service | `SetDownloadPrefix(conf.DownloadDirectory)`

                - call the after action | `initAction += conf.AfterLibCoreStarted;` | set the event signal 

                   ```c#
                   Action ac = () =>
                   {
                       loggerInitAndShutdown.Info("InitLibCore() done."); 
                       _waitEvent.Set();
                   };
                   ```

              3. wait event   wait the libcore signal

              4. setup fec server  "it likes **Fec2** above", the Fec2 is not running.

                - setup rpc servcer 

                  ```c#
                  PInvokeProxy.SetupRpcServer(
                      config.FecTransportEnabled, 
                      "127.0.0.1", 
                      config.FecTransportRpcServerPort);
                  ```

                  

                - generate the FecServerStarter then start the service

                  ```c#
                  internal void Start(string instanceIndex)
                  {
                      if (DockerEnable)
                      {
                          CheckDocker();
                          StartDocker();
                          return;
                      }
                      else
                      {
                          if (SystemConfigurationInfo.IsWindows)
                          {
                              _taskHost = new GenericProcessInvokerQueue();
                              _taskHost.Invoke(BinName, _args, 1, false);
                          }
                          else
                          {
                              string newExecStart = $"{Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "TVUFecServer")} {_args}";
                              _systemdService = new SystemdFileItem(
                                  $"tvu.fecserver{instanceIndex}", 
                                  $"TVU FecServer for R#{instanceIndex}", 
                                  newExecStart, 
                                  AppDomain.CurrentDomain.BaseDirectory, 
                                  null, 
                                  true, 
                                  1);
                              _systemdService.AddUnitFile();
                              ServiceItem.ReloadDaemon();
                              _systemdService.Restart();
                          }
                      }
                  }
                  ```

            - initialize receiver information `ReceiverInfo.Instance.Init(RX.PInvokeProxy);`

            - conditional on start router service `RX?.StartRouterServer();`

              1. start router service  `"RouterServer", "TVURouter"`   |  only windows, the router is VLAN plugin on Linux

            - initialize the thumbnail module `ThumbnailProxy.Instance.Init(ReceiverInfo.Instance, timeSpan, true, DownloadPeerThumbnail, GrpcServerProxy.Instance.GrpcPort);`

            - conditionally config the msgpipe  `MsgPipeClientProxy.Instance.ConfigClientWithLibCore(ReceiverInfo.Instance.StrId, null);`   |   conditional is `if (!SilentMode)`

              1. generate the `IMsgPipeClient`   `_client = new RMsgPipeClient(peerId, msgTypes);`

            - get device information and initialize the VoIP module  then start it

              ```c#
              VoIPBtnInfoUpdater.Instance.Init(
                  RX.ReceiverID, 
                  RX.InstanceIndex, 
                  GrpcServerProxy.Instance.GrpcPort, 
                  OnVoIPError, 
                  OnVolumeUpdated, 
                  device.Recording.FirstOrDefault(), 
                  device.Playback.FirstOrDefault());
              ```

              1. start PeerClientVoIPInvoker   `Path.Combine(SystemConfigurationInfo.IsWindows ? "VoIP" : "WebRTC", "peerclient");`
              2. using httpclient to get the VoIPIDList, and the url is `string.Format(ModuleConfig.Instance.VoIPGetSessionsUrl, _rPeerIdString)`

            - gerenate `LiveBeforePairedManger` then initialize it

              1. bind timer event `_tryToLiveInterval.Elapsed += _tryToLiveInterval_Elapsed;`

         3. LibcoreControl start `RX?.StartLibCore();`

            - start receiver  `PInvokeProxy.StartReceiver(OnStartRErrorOccured);`

              1. send message to libcore service

                 ```c#
                 StartReceiver(
                     InitConf.PeerServiceHost, 
                     (ushort)InitConf.PeerServicePort,
                     InitConf.ExternalIP, 
                     (ushort)InitConf.ExternalPort,
                     InitConf.LocalIP, 
                     (ushort)InitConf.LocalPort,
                     InitConf.RUrl, 
                     InitConf.RPlayShmUrl,
                     onErrorOccured);
                 [LibCoreAPIInit]
                 public void StartReceiver(string rPSIP, ushort rPSPort, string rExternalIP, ushort rExternalPort, string rLocalIP, ushort rLocalPort, string rUrl, string rPlayShmUrl, Action<int, string> onErrorOccured)
                 {
                     logger.Debug($"WrapLibCore StartReceiver: ");
                     P1.P1.StartReceiver(rPSIP, rPSPort, rExternalIP, rExternalPort, rLocalIP, rLocalPort, 4, 0, rUrl, rPlayShmUrl);
                 }
                 ```

         4. generate MediaMindController then initialize

            - bind event  `PlaybackModel.Instance.PlaybackStopped += CleanStory;`

         5. initialize partyline manager |  load tml file and initialize configuration.  `EP2 = Load(Path.Combine(encodingProfile2Path, "PartylineVFB.tml"));`

         6. generate the NginxProxy then initialize it  `NginxProxy?.Init(ReceiverInfo.Instance.StrId);`

            ```c#
            AppCore.Instance.MeetStartMilestone("Initializing nginx.");
            AppCore.Instance.NginxProxy?.UpdateNginxConf(
                "WebSocket.conf", 
                (CoreRuntimeStatus.PortMappings.Instance.WebAPI.LocalPort + 10).ToString(), 
                @"proxy_pass http:\/\/127\.0\.0\.1:(?<port>\d+?)\/ws;", 
                "proxy_pass http://127.0.0.1:{0}/ws;");
            AppCore.Instance.NginxProxy?.UpdateNginxConf(
                "WebRNginx.conf", 
                (CoreRuntimeStatus.PortMappings.Instance.WebAPI.LocalPort).ToString(), 
                @"proxy_pass http:\/\/127\.0\.0\.1:(?<port>\d+?)\/;", 
                "proxy_pass http://127.0.0.1:{0}/;");
            UpdatePeerIDs(receiverID);
            UpdateVideoConf(receiverID);
            ```

         7. grpc servers initialize `GrpcServerProxy.Instance.Init();`

            - conditionally generate and initialize `SCTESwitcher`    |   **TODO**: need to add detail information

            - generic grpc server initialize `_grpcServer.Init(ModuleConfig.Instance.FilterGrpcPort);`

              1. generate grpc server with the specified port then start the grpc server

              ```c#
              public void Init(int port)
              {
                  server = new Server
                  {
                      Services = { ChildProcessStatus.BindService(impl) },
                      Ports = { new ServerPort("127.0.0.1", port, ServerCredentials.Insecure) }
                  };
                  if (server != null)
                      server.Start();
                  logger.Info("Grpc client is listening port {0} for gRPC.", port);
              }
              ```

            - conditionally initialize `VirtualClipGrpcServer`  |  the process is similar to the generic grpc server

            - conditionally initialize `CloudDecoderGrpcServer` |  the process is also similar to the generic grpc server

            - conditionally initialize `SCTE104GrpcServer`  |  the process is also similar to the generic grpc server

            - bind the delegate

              ```c#
              GrpcServer.ReportVoIPStatusInfo += VoIPBtnInfoUpdater.Instance.OnReportVoIPStatusInfo;
              GrpcServer.ReportVoIPVolumeInfo += VoIPBtnInfoUpdater.Instance.OnReportVoIPVolumeInfo;
              ```

         8. load entity configuration  `EntitySettings.Load(Path.Combine(Conf7Path, "EntitySettings", "EntitySettings.tml"));`

         9. generate the `TVUPluginHostManager` then initialize it. `_pluginHostManager?.Init(pluginEntryPoint);`

            - sync the settings

              1. load the `/Plugin/Plugins_Console.json`

              2. merge the plugins  `knownPlugins = pluginList.MergePlugins();`

              3. sync the `knownPlugins` from `pluginEntryPoint.SyncSettingItems`

              4. conditionally use the `RFeaturesManager` and initialize it.

                 - get the authorized feature information from local file `wastenomoretime.dat` or libcore service

                 - sync the `FeatureSwitch.db` file base on `pluginEntryPoint`,  if miss in the file then add to file, if miss in the  `pluginEntryPoint` then add it in the file.

                 - check the authorized  feature from above features

                 - fill feature extra 

                   1. load feature extra from `ExtraFeatureMapping_Console.db`

                   2. combine `pluginEntryPoint.FeatureMappings` with load feature extra 

                   3. set the related feature of feature extra 

                      ```c#
                      List<FeatureEx> mappings = InitMapping(pluginEntryPoint);
                      foreach (FeatureEx featureEx in mappings)
                      {
                          _dictFeatureEx.Add(featureEx.TargetName, featureEx);
                          FeatureSwitch feature;
                          if (AuthorizedSupportedFeatures.TryGetValue(featureEx.FeatureName, out feature) 
                              && feature != null)
                          {
                              featureEx.RelatedFeature = feature;
                          }
                      }
                      ```

              5. conditionally  sync feature control

                 - get authorized feature extra with the `DllPath` property of `TVUPluginInfo` in the  `knownPlugins`  then set the `RelatedPlugin`  |  `featureEx.RelatedPlugin = plugin;`
                 - get the `FeatureSwitch` from `RelatedFeature` of  feature extra  |  `FeatureSwitch feature = featureEx.RelatedFeature;` 
                 - set the `TVUPluginInfo.Enabled`   with the `FeatureSwitch.IsOpen`

              6. write the filter plugin object to the `/Plugin/Plugins_Console.json`

         10. register default share memory

             - get the default **SHM** object from `PlaybackModel`,  the default SHM name is 

               ```c#
               vSHMName = $"BYPASSV{SwitcherIndex}";
               aSHMName = $"BYPASSA{SwitcherIndex}";
               ```

             - generate the EncoderSharedMemory `new EncoderSharedMemory("Default", Player.DefaultSHM.VSHMName, Player.DefaultSHM.ASHMName, true);`

             - register the `EncoderSharedMemory `   |  `EncoderSourceManager.RegisterOne(playbackSharedMemory);`

             - set the default shared memory `EncoderSourceManager.SetDefaultSharedMemory(playbackSharedMemory);`

         11. check if open `OEMLogo` feature `bool isOEMLogoEnabled = FeatureManager?.TryGetFeatureOpenedByName(RFeatureConst.FEATURE_OEM_LOGO) ?? false;`

         12. initialize the `PlaybackModel`

             ```c#
             Player.Init(
             RX.InstanceIndex, 
             ReceiverInfo.Instance, 
             GrpcServerProxy.Instance.GrpcPort, 
             isOEMLogoEnabled, 
             HWInfo.SDICardVendor, 
             K.CurrentOutputFormat2,
             Recipe?.RunSwitcher ?? true,
             HWInfo.SupportHWDecodingForH264, 
             HWInfo.SupportHWDecodingForHEVC);
             ```

             - set the capture passthrough mode  `DecklinkHelper.SetCapturePassthroughMode();`

               1. start `decklink_capture_passthrough_mode` process and set the arguments

                  ```c#
                  Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "iMatrix", "decklink_capture_passthrough_mode");
                  ...
                  process.StartInfo.Arguments = "0";
                  ...
                  process.Start();
                  ```

             - conditionally generate the `FastSwitch` then initialize it   only **Windows**

               1. start a new thread to run a action, block the thread to wait a event

                  > when calling the `TrySwitch` method and set the `IPeerEntity`,  the event signaled and execute the `playbackModel.SwitchChannel(_peer.PlaybackSkeleton);`

             - generate the `SwitcherProxy`   |  `new SwitcherProxy(ModuleConfig.Instance.SwitcherPort, runSwitcherProcess);`

             - conditionally generate the `SwitcherInvoker2` 

               ```c#
               matrixSwitcher = new SwitcherInvoker2(
                   isOEMLogoEnabled, 
                   vendor, 
                   outputFormat.SwitcherName,     
                   supportHWDecodingForH264, 
                   supportHWDecodingForHEVC);
               ```

               1. set the `Switcher` arguments
               2. generate the `iMatrixPlayer's imatrixcfg.xml` from `Receiver's Config.xml ` `Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "iMatrix", "imatrixcfg.xml");`

             - call `matrixSwitcher.Start(instanceIndex, ConnectToSwitcherOnly);`

               1. kill process `switcher, playeroftvu, proxy_server, ns`

               2. start `swithcer` process  | `Path.Combine("iMatrix", "switcher");`

               3. circularly execute the action `PlaybackModel.ConnectToSwitcherOnly` until set the switcher successfully

                  - visit `switcher.GetCurrentInputs`  and delete the original channels
                  - switch the channel to last channel  `Init_SwitchToLast();`
                  - update the player version  `_switcher.GetVersion();`
                  - set switcher logo `InitSwitcherLogo();`
                  - set R information to switcher `Init_SetSwitcher();`
                    1. `_switcher.SetRInfo(ID, Name, AIPort);`
                  - `CheckAndSetOemLogosInternal();`

               4. conditionally start `AIService`   only **Linux**| `StartAIService(ReceiverInfo.Instance.DifferentInstance, PlaybackModel.Instance.AIPort);`

                  - set the binary name `$"{Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "iMatrix", "aiproxy")} -p {aIPort}";`

                  - start the process with `docker` or `systemd`

                    > docker image name is `ai_collection:v1`

             - register the `CentralManagementMainMessagePump`  |  `CentralManagementMainMessagePump.Instance.Register(new CentralManagementProxy());`

         13. initialize `PlaybackHelper`   `_playerHelper.Init();`

             - bind `KernelState.CurrentPeerChanging`   |  call `AppCore.Instance.Player?.StopOutput(currentPeer.PlaybackSkeleton);` or `AppCore.Instance.Player?.Stop(currentPeer.PlaybackSkeleton, true);`
             - bind `KernelState.CurrentPeerChanged`  |  select T  `AppCore.Instance.Player?.SelectT(currentPeer);`
             - bind `KernelState.PropertyChanged` | when the property is `CurrentTVersion`, execute `PlaybackHelper.TryUpdateLivePackPeer();`
             - bind `KernelState.settingChanged` | when the setting item is `CurrentTVersion`, execute `PlaybackHelper.TryUpdateLivePackPeer();`
             - bind `ReceiverInfo.Instance.RNameChanged` | when the receiver name changed, call `AppCore.Instance.Player?.ChangeName(newName);` to set receiver information  `_switcher?.SetRInfo(ID, newName, AIPort);`
             - bind `AppCore.Instance.Player.KeyFrameRequested`  | if current peer is `Pack` , call `peer.Refresh();`

         14. conditionally generate the `ASIOutputController` then initialize it   `ASIOutput.Init(MultimediaSupport.ModuleConfig.Instance.Outputcardid, Player.DefaultSHM);`

             - load the configuration `Path.Combine(AppCore.Instance.EncodingProfile2Path, "ASIOutput.tml")` in construction function 

             - generate the encoder `TVU264` | `Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "ExEncoder30", "TVU264.exe");`

             - update the `encoder ` parameters

               ```c#
               Encoder = new TVU264(DependencyInfo.EncoderPath, string.Empty);
               TVU264ParameterHelper.UpdateGlobalParameters(Encoder, Path.Combine("log", $"ASIOutput_port{cardID}.log"));
               TVU264ParameterHelper.UpdateInputParameters(Encoder, shm.VSHMName, shm.ASHMName);
               bool deinterlace = false;
               bool.TryParse(EP2.Video.Deinterlace, out deinterlace);
               uint vBitrate = 0;
               TVU264ParameterHelper.UpdateVideoCodecParameters(
                   Encoder, 
                   EP2.Video.Codec, 
                   EP2.Video.Resolution, 
                   EP2.Video.Bitrate, 
                   ref vBitrate, 
                   EP2.Video.GOP ?? VideoEncodingProfile2.DefaultGOP, 
                   deinterlace, 
                   EnumResolutionCalculationStrategy.KEEP_ASPECT_WITH_FIXED_WIDTH, 
                   EP2.Video.Scale, 
                   EP2.Video.FrameRate, 
                   isCbr: true);
               EncoderHelper2.UpdateAudioCodec(Encoder, EP2.Audio);
               string outputUrl = Helper.GetASIUrl(cardID.ToString());
               TVU264ParameterHelper.UpdateOutputParameters(Encoder, "mpegts", outputUrl);
               ```

               

             - start `tvu264` process  |  `Encoder?.Invoke();`   using process or `systemd` to start process

         15. initialize `S3TransferClientProxy`  `S3TransferClientProxy.Instance.Init();`

             - generate the grpc `Channel` and pass it into `TVUS3TransferService.TVUS3TransferServiceClient`
             - initialize download client  `InitDownloadClient();`
             - initialize upload client `InitUploadClient();`

         16. initialize `HDRController`  `HDRController.Instance.Init(MultimediaSupport.ModuleConfig.Instance.PixelFormat);`  |   set the default pixel format

         17. load plugins `_pluginHostManager?.Load();`

             - bind event `AppDomain.CurrentDomain.AssemblyResolve`

             - load plugins `TVUPluginHost.LoadPlugins`  |`_pluginHost.LoadPlugins(PluginEntryPoint.PluginFilePath, PluginEntryPoint.BasicPlugins);`

               1. deserialize `Plugins_Console.json` to `TVUPluginList`  |  `TVUPluginList pluginManager = TVUJsonSerializable2.DeserializeObjectFromFile<TVUPluginList>(fileName);`

               2. conditionally use basic plugins| when the `pluginManager` is null

               3. select all plugin assembly files and load these file in our system, **but don't initialize them**.  `unsortedPlugins = LoadPlugins(listPlugins)  ->  LoadPlugin(assemblyName);`

                  - find the entry point, the entry  point is a class and the class name is `EntryPoint` , the module name is similar to `TVU.OptionalComponent.Booking2_v15` , and the module name must end with `_v15`

                    ```c#
                    WellKnownTypeNames.EntryPoint = "EntryPoint";
                    ...
                    entryPointQualifiedName = string.Format(
                        "{0}.{1}", 
                        assembly.ManifestModule.Name.Substring(0, assembly.ManifestModule.Name.Length - 4), //move the `_v15`
                        WellKnownTypeNames.EntryPoint);
                    ```

                  - the `EntryPoint` class must inherit `TVU.Plugin.Common.ITVUPlugin`

                  - generate the `EntryPoint` instance and convert the type to `TVU.Plugin.Common.ITVUPlugin`

                  - add the plugin object in to dictionary `_dictLoadedPlugins.Add(assemblyName, plugin);`

                  >All plugins must be loaded successfully
                  >
                  >`Debug.Assert(success == assemblyNames.Count, "Not all plugins loaded successfully.");`

               4. sort all loaded plugins `SortPlugins(unsortedPlugins);`

                  - wrapper the plugin to `PluginSortWrapper`   `listSortWrapper.Add(new PluginSortWrapper(plugin, count++));`
                  - sort the plugins `listSortWrapper.Sort(StableSort);`
                    1. `AbstractTVUPlugin` has implemented the `IComparable<ITVUPlugin>`  and the default return value is `0`
                    2. if the compare value is `0`,  it will use the `PluginSortWrapper.Index`   |   the index is the count that passed into the construction.
                    3. return the sorted plugins

               5. initialize all plugins `InitPlugins(LoadedPlugins);`

                  - execute the `ITVUPlugin.Init()`
                  - check if the plugin failed to initialize and print the failed plugins.
                  - all plugins must be initialized successfully.`Debug.Assert(success == plugins.Count, "Not all plugins inited successfully.", failedDetails);`

             - if load fails, send the report to `Raven` |  `RavenClientProxy.Instance.SendReport(...)`

             - execute the receiver initialized action `WellKnownEvents.ReceiverInited?.Invoke();`

               > if a plugin is interested on the receiver initialized, it can subscribe to the delegate.

         18. reload Nginx `NginxProxy?.ReloadAtInit();`

             - if the system is the **Windows**, directly start the process  `process.Start();`
             - else the system is **Linux**, directly the service  `_systemdServiceNginx?.Restart();`

         19. conditionally initialize `MsgPipeClientProxy`   `MsgPipeClientProxy.Instance.Init();`   |   conditional is `if (!SilentMode)`

             - call `RMsgPipeClient.Init` method |  `_client?.Init(onMessageReceived, null);`

               1. send the initialization command to libcore service  `AppCore.Instance.RX.PInvokeProxy.P1.P1.InitMsgPipe(msgTypes, true);`

               2. get the message from libcore service. `AppCore.Instance.RX.PInvokeProxy.P1.P1.GetMsg(OnMessage);`   |  using **async** grpc method

                  >the **OnMessage** method will execute the `OnMessageReceived` when getting the message from libcore service

         20. execute the finished action `onFinished?.Invoke();`  |   the action location is `ConsoleR.OnAppOnLoadedFinished`

             - execute the initialized last logic 

               1. execute the `DelayedInit` action   `WellKnownEvents.DelayedInit?.Invoke();`

                  > if a plugin wants to execute some logics after initialization stages, it can subscribe the action

               2. initialize the `SourceListManager`  | `SourceListManager.Instance.Init(ChangePeer);`

                  - bind the `SelectedPeerChanged` action  |  `SelectedPeerChanged += onSelectedPeerChanged;`   

                    > the `onSelectedPeerChanged` is the `AppCore.ChangePeer` method

                  - initialize each known source.  `SourceListManager.Instance.Init(ChangePeer);`

                    ```c#
                    public void Init(Action<IPeerEntity, bool, Dictionary<string, object>> onSelectedPeerChanged)
                    {      
                        SelectedPeerChanged += onSelectedPeerChanged;
                    
                        foreach (ITVUSourceList sourceList in KnownSourceLists.Values)
                            sourceList.Init();
                    }
                    ```

                    

                    > The **plugin** can call the `SourceListManager.RegisterSourceList(ITVUSourceList newList)` to register source list.
                    >
                    > Then the source list object can be initialized when call  `SourceListManager.Init(Action<IPeerEntity, bool, Dictionary<string, object>> onSelectedPeerChanged)`

                  - initialize `AppCoreLastCmdHandle`  `AppCoreLastCmdHandle.Instance.Init();`

                  - generate the `AppCoreLastErrorHandle` then initialize it `new AppCoreLastErrorHandle(); ... _lastErrorHandler?.Init();`

                  - register the central management proxy `CentralManagementMainMessagePump.Instance.Register(new CentralManagementProxy());`

                    > `CentralManagementMainMessagePump` hold all registered `ICentralManagedModule` .
                    >
                    > register module via the `CentralManagementMainMessagePump.Register` method 
                    >
                    > pump the message from `CentralManagementMainMessagePump.PumpMessage(GenericLiveSwitchMessage msg)`, select the module from the `ICentralManagedModule.CategoryId` then call the `ICentralManagedModule.HandleMessage`.
                    >
                    > the `PumpMessage` method is called by `CommandCenterProxy.HandleModernCategory`
                    >
                    > `CommandCenterProxy` generate the `TVUWebSocketClientProxy` , then register the `CommandCenterProxy.OnMessageReceived` and receive the web socket mesage

                  - register `APIOperation`  `TransportRRemoteOperation.CoreOpreationReceived += APIProxy.Instance.APIOperation;`

                    > `TransportRRemoteOperation.SharedRemoteOperation`  will be called by `TVUNancyModule.HandlePost`.
                    >
                    > the http routes are defined in `NancyModule`
                    >
                    > the `Nancy` started in aspnet core
                    >
                    > the `Nancy` is hosted by `TVUNancyFxHost`, and `TVUNancyFxHost` started in `EntryPoint` of `TVU.OptionalComponent.WebSiteHost2_v15`  module 

                  - register the web socket session to handle the web socket message  `WebsocketSessions.RemoteOperations[CentralManagementProxy._categoryId] = APIProxy.Instance.APIOperation;`

                    > the web socket server is started in `ConsoleR.Run`   7th step

                  - conditional `if (!SilentMode)` 

                    1. get command center configuration 

                    2. format the server URL and reset it  `CommandCenterProxy.Instance.ResetServerUrl(ccUrl);`  

                    3. initialize `CommandCenterProxy`  `CommandCenterProxy.Instance.Init(ReceiverInfo.Instance.StrId, GetInitMessage, GetExitMessage);`

                       - generate the `TVUWebSocketClientProxy`
                       - register client `RegisterClient`

                       ```c#
                       Proxy = new TVUWebSocketClientProxy();
                       Proxy?.RegisterClient(
                           ClientID, 
                           url, 
                           getInitMsg, 
                           OnMessageReceived, //this is raise the CentralManagementMainMessagePump.Instance.PumpMessage
                           getExitMsg, 
                           ModuleConfig.Instance.CommandCenterWSIsCheckStatus, 
                           ModuleConfig.Instance.CommandCenterWSCheckStatusInterval, 
                           ModuleConfig.Instance.CommandCenterWSStatusTimeoutSeconds);
                       ```

                       

                  - conditionally generate the `LiveOnHoldStatusChecker`  then initialize it  `_liveOnHoldStatusChecker?.Init(ModuleConfig.Instance.LiveOnHoldTimeoutSeconds);`

                    > `LiveOnHoldStatusChecker`  is will start a new thread to execute the `LiveOnHoldStatusChecker.DoWork`

                  - start the timer `StartTimer();`   decrease the `KernelState.WaitingSeconds`

             - unsubscribe the `StartMilestoneMeet` action |   `AppCore.Instance.StartMilestoneMeet -= OnStartMilestoneMeet;`

             - unsubscribe the `NotificationAddedaction`  action |   `NotificationCenterModel.Instance.NotificationAdded -= OnAddNotification;`

         21. if catch the exception, will report the exception to `Raven`   `RavenClientProxy.Instance.SendException(ex);`

    7. web socket server initialization  `ReceiverRuntime.WebsocketServer.Program.Init(args);`

       - using AspNet core to start the web server

       - initialize logic in the `Startup` file

         1. generate the `WebsocketHandle` in `ConfigureServices`

         2. add new action in `WebsocketSessions.RemoteOperations`

            ```c#
            WebsocketSessions.RemoteOperations[1] = MultimediaSupport.WebAPI.APIProxy.Instance.APIOperation;
            WebsocketSessions.RemoteOperations[2] = CoreComponent.Thumbnail2.APIProxy.Instance.APIOperation;
            ```

         3. inject a new service  `services.AddTransient<SendMessages, SendMessages>();`

         4. add web socket middleware `app.UseWebSockets(webSocketOptions);`

         5. add a anonymous middleware to intercept a specified path  `if (context.Request.Path == "/ws")`

         6. handle the web socket  `WebSocket webSocket = await context.WebSockets.AcceptWebSocketAsync();   await _handle.Echo(webSocket);`

            - check the connected web socket count, if the count >100, close the web socket

            - get the message from web socket `webSocket.ReceiveAsync`
            - deserialize the message to `RemoteOperationRequest`
            - select the `RemoteOperationHandler` from `WebsocketSessions.RemoteOperations` `WebsocketSessions.RemoteOperations.TryGetValue(body.CategoryId, out handler);`
            - handle the request message and send the return value of handler

       > main thread directly start the web socket server, so the thread holds on the asp net core process and doesn't execute the next step. 

    8. wait shutdown signal


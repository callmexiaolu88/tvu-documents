- need administrator to debug, because some p/invoke needs permission.  DONE
- need to resolve the assembly loading, so register the `AppDomain.CurrentDomain.AssemblyResolve` event   c   DONE
- `TVU.CoreComponent.Entity2.LibPeerID.DeviceData.RestoreInfoStoragePath = "/data/restore/RestoredReceiverInfo.json"` conditional DONE

- lack TVU.ThumbnailCreation.Creator3   s

- `TVU.CoreComponent.LibCoreProxy_v15`  NugetPackage `NAudio 1.8.5`(don't support `dotnet core`) , upgrade `1.9.0`   DONE

- `GLinkEncoder` lacks Assembly information c    DONE

- TVU.Plugin.Receiver.Common.PInvoke2.DllName  `@"..\iMatrix_x86\libbypassstream.dll"` ->`@".\iMatrix\libbypassstream.dll" ` s   DONE

- `TVU.OptionalComponent.ExternalEncoder.SwitcherProxy` is null, because `moduleConfig.IsSeamlessSwitchEnabled` is false. When Constructing `WebRTCPreviewWorker`, the `shm` is null, so throw null reference.   todo

  `if check the conditioanl, and don't set the _previewWorker3, the problem will be raised below `

  - VolumePreview(IEncoderSource2 shm)  shm is null

- TVU.OptionalComponent.GridEncoder.CommandV2.Equals  null reference exception, because `property.GetValue(obj, null)` is null, but call Equal method  s

- `dcap2` throw exception, because of  lack of dependency, and these dependencies are in the `iMatrix_x86`    (refer to last item)

  - avcodec-55.dll / libstdc++-6.dll / libgcc_s_dw2-1.dll/libwinpthread-1.dll(x86)

- `TVU.App.ReceiverRuntime.TVUPluginHostManager` line 72, suggest to remove the comment or . Because of the `notFoundLib ` can be modify, when `AssemblyResolve` event is fired. s

- **dcap2** needs `libwinpthread-1.dll(x86)  `, **TVU264** needs `libwinpthread-1.dll(x64)`

http://localhost/R/WebRWebSite/WebR/vue-project/dist/index.html



TVU.App.ReceiverRuntime.DependencyInfo.CapturerPath

Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "ExEncoder30", "dcap2")

Path.Combine(AppDomain.CurrentDomain.BaseDirectory, SystemConfigurationInfo.IsWindows ? "iMatrix_x86" : "ExEncoder30", "dcap2")
Widevine
############




概要
=========

源文件使用content\_key加密后，通过CDN发布

客户端从CDN下载加密的文件内容，向License Server请求License以获取content\_key相关信息，解密文件

License内容可参考 `transcode_with_drm.json <https://gist.github.com/gaberussell/016705e01014127f74a833e9c3e41b5c>`_

Key
========

DRM Key -> Device Key -> Content Key，层层保护

设备在出厂时可以有一个uniq device key，用于加密content key。设备请求时，通过uniq device key id或uniq device id、content id向license server请求encrypted content key。


参考资料
===========

- `DRMへの対処法－パート1：DRMに対する理解と保護されたコンテンツの生成方法 <https://www.brightcove.com/ja/blog/2017/01/dealing-drm-understanding-drm-and-how-produce-protected-content>`_
- `Widevine CDM Considerations: Why Streaming Services Should Encourage Browser Updating <https://castlabs.com/news/widevine-cdm-browser-updating/>`_
- `Playing DRM Content <http://webostv.developer.lge.com/develop/app-developer-guide/playing-drm-content/>`_
- `widevine-license-token-request.md <https://github.com/AdobeDocs/primetime.en/blob/master/help/digital-rights-management/multi-drm-workflows/license-token-req-resp-ref/widevine-license-token-request.md>`_
- `Widevine license token request / response <https://github.com/AdobeDocs/primetime.en/blob/master/help/digital-rights-management/multi-drm-workflows/license-token-req-resp-ref/widevine-license-token-request.md>`_
- `Encrypting Videos for Streaming: AES and DRM Systems <https://medium.com/@sourabhmodi20051/encrypting-videos-for-streaming-aes-and-drm-systems-1249bdb2a5b5>`_
- `ENCRYPTION KEY PROVISION <https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-key-provision.html>`_
- `Widevine DRM and Keybox <https://www.programering.com/a/MDO1EjNwATQ.html>`_
- `drm system <https://www.intertrust.com/products/drm-system/developer/restapi/>`_
- `The Many Facades of DRM <http://www.whiteboxcrypto.com/files/2012_MISC_DRM.pdf>`_
- `Widevine Level 1 Provisioning Models. version 1.2 <https://docplayer.net/91605271-Widevine-level-1-provisioning-models-version-1-2.html>`_

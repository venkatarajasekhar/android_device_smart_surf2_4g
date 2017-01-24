## [ROM][UNOFFICIAL][6.0.1] CyanogenMod 13 for MTC Smart Surf2 4G ## (Decker, [http://www.decker.su](http://www.decker.su))

Сборка для запуска на стоковом ядрке 3.18.19 (!), 32-bit.

Внимание! Это тестовое дерево для сборки CM13.0 для МТС Smart Surf2 4G. Работы над деревом ведутся в данный момент,
поэтому прошивка может не собираться / не запускаться и т.п. 

- https://github.com/Nonta72/android_device_d5110_infinix/tree/cm-13.0 - дерево построено на основе этого CyanogenMod 13 for HOT 2.
- https://forum.xda-developers.com/hot-2/development/rom-cyanogenmod-13-t3442726

Заметки (Decker)
----------------

    ERROR: /home/decker/cm-13.0/frameworks/opt/telephony/../../../device/infinix/d5110/ril/telephony/java/com/android/internal/telephony/MediaTekRIL.java:299: Operators cannot be resolved to a type
    ERROR: /home/decker/cm-13.0/frameworks/opt/telephony/../../../device/infinix/d5110/ril/telephony/java/com/android/internal/telephony/MediaTekRIL.java:299: Operators cannot be resolved to a type
    ERROR: /home/decker/cm-13.0/frameworks/opt/telephony/../../../device/infinix/d5110/ril/telephony/java/com/android/internal/telephony/MediaTekRIL.java:828: Operators cannot be resolved to a type
    ERROR: /home/decker/cm-13.0/frameworks/opt/telephony/../../../device/infinix/d5110/ril/telephony/java/com/android/internal/telephony/MediaTekRIL.java:828: Operators cannot be resolved to a type

device/infinix/d5110/ril/telephony/java/com/android/internal/telephony/MediaTekRIL.java.bak 

                if (strings[i] != null && (strings[i].equals("") || strings[i].equals(strings[i+2]))) {
    [!]		Operators init = new Operators ();
    		String temp = init.unOptimizedOperatorReplace(strings[i+2]);
    		riljLog("lookup RIL responseOperatorInfos() " + strings[i+2] + " gave " + temp);
                    strings[i] = temp;
                    strings[i+1] = temp;
                }


MediaTekRIL.java и gps.h временно убраны из дерева (добавлено расширение .bak) чтобы убрать проблемы при сборке.
Отсутствующий Operators.java необходимый для сборки MediaTekRIL.java можно посмотреть здесь:
https://github.com/SeriniTY320/android_device_THL4000-cm-13.0/blob/master/ril/telephony/java/com/android/internal/telephony/Operators.java

libmtkplayer необходимый для FM Radio пока отключен в Vendor'е.

При сборке применен патч /home/decker/cm-13.0/external/sepolicy для поддержки POLICYVERS := 29, содержимое
/home/decker/cm-13.0/external/sepolicy которое использовалось при сборке есть в архиве external_sepolicy.tar

Что работает, что не работает
-----------------------------

[1] На данный момент работает запуск прошивки и отображение графики. Все остальное можно сказать не работает, т.е.
*не работает*:

WiFi, RIL, звук, камера и т.п.

Надо смотреть что сделано в дереве от CM14 и пробовать перенести изменения сюда.

Лог: 

logcat2.txt:




    01-04 09:08:52.925   294   294 I MtkAgpsNative: [CDMA mgr] cdma_mgr_dlopen_firmware 
    01-04 09:08:52.925   294   294 E MtkAgpsNative: [CDMA mgr] VIAMTKDBG Could not open /system/lib/libviagpsrpc.so, error: dlopen failed: library "/system/lib/libviagpsrpc.so" not found
    01-04 09:08:52.925   294   294 D agps    : [agps] WARNING: [MAIN] cdma_mgr_dlopen_firmware failed
    01-04 09:08:59.948   284   284 E HAL     : dlopen failed: cannot locate symbol "_ZN7android11AudioSystem24getVoiceUnlockDLInstanceEv" referenced by "/system/lib/hw/audio.primary.mt6737m.so"...
    01-04 09:09:07.498   284   284 E MtkOmxCore: dlopen failed, dlopen failed: library "libMtkOmxWmaDec.so" not found
    01-04 09:09:07.499   284   284 E MtkOmxCore: dlopen failed, dlopen failed: library "libMtkOmxWmaProDec.so" not found
    01-04 09:09:07.566   284   284 D MtkOmxVdecEx: [0xb0822000] ClearMotion is not enabled. dlopen failed: library "/system/lib/libClearMotionFW.so" not found

[2] С последним коммиторм на момент 1/21/2017 6:59:06 PM  в данной прошивке уже работают WiFi, звук, камера (съемка фото, съемка видео пока не работает из-за проблем с кодеками, но причина тут уже другая, не та которая в дереве от CM14.1, т.е. здесь данная проблема наверняка решится быстрее ... с xml медиакодеков от CM14.1 поведение камеры при записи видео было точно такое же, как и в CM14.1, т.е. после 1-й секунды записи был звук остановки видео, здесь же после [этого коммита](https://github.com/DeckerSU/android_device_smart_surf2_4g/commit/9d90dd6ea64804119cf179bf0d91dbc12d496449) уже другая ситуация, после двух секунд записи падает камера ... и эту проблему уже вроде бы решали, пока нет времени окончательно разобраться), также работает GPS и запись с экрана. Но пока не удалось завести RIL ... Т.е. "Прошивка модуля связи" **неизвестно**. 

В данный момент идет работа над запуском RIL'а.

[3] RIL:

lib
---
libmal.so
libmdfx.so
librilmtk.so
librilmtkmd2.so
mtk-ril.so
mtk-rilmd2.so

bin
---

gsm0710muxd
gsm0710muxdmd2
mtkrild
mtkrildmd2

divis1969: Если вы можете собирать и анализировать логи, то последовательность запуска телефонии на mt6735 примерно такая
1. Стартует ccci_mdinit, его задача - загрузить в модем фирмваре
2. После него стартует gsm0710muxd, он подготавливает несколько последовательных каналов (tty) для работы с модемом
3. Потом стартует ril-daemon-mtk, он как раз и пользуется этими каналами, а сам создает сокеты для Java RIL
4. Стартует Java RIL и подключается к сокетам. В логе можно искать по тегу RILJ

https://habrahabr.ru/post/183984/

[4] FIX RIL:

BIN
gsm0710muxd
gsm0710muxdmd2
mtkrild
mtkrildmd2

ETC
firmware (folder)
mddb (folder)
apns-conf.xml
spn-conf.xml
trustzone.bin
LIB & LIB64
libc2kril.so
libc2kutils.so
libreference-ril.so
libril.so
librilmtk.so
librilmtkmd2.so
librilutils.so
libviatelecom-withuim-ril.so
mtk-ril.so
mtk-rilmd2.so
volte_imsm.so

# logcat --help
...
  -b <buffer>     Request alternate ring buffer, 'main', 'system', 'radio',
                  'events', 'crash' or 'all'. Multiple -b parameters are
                  allowed and results are interleaved. The default is
                  -b main -b system -b crash.
...

logcat -b radio (!)
logcat -b all (!)

Файлы относящиеся к RIL пока берутся из вендора, но могут быть и скомпиленными из исходников, для этого
достаточно раскомментировать:

#PRODUCT_PACKAGES += \
#    gsm0710muxd \
#    gsm0710muxdmd2 \
#    mtkrild \
#    mtkrildmd2 \
#    mtk-ril \
#    mtk-rilmd2

[5] Удалось достичь некоторых успехов. С применным патчем 0002-xen0n-some-MTK-services-e.g.-ril-daemon-mtk-require-.patch
(правда пока непонятно, с blob'ами со стока или собранными из исходников) ril-daemon запустился, /dev/socket/rild
создался и Baseband Firmware и IMEI отобразился. Направление верное, т.е. одна из проблем была в нехватке переменных
окружения под названия сокетов создаваемых ril-daemon-mtk ...

[6] При изменении конфигурации, например, языковых пакетов делается make installclean ... Просто напоминалка.

Полезные ссылки
---------------

Деревья для сборки CM13:

- https://github.com/ferhung-mtk/android_device_Xiaomi_HM2014011 - 6582
- https://github.com/Nonta72/android_device_d5110_infinix/tree/cm-13.0 - 6580 (3.18.19)
- https://github.com/jsharma44/android_device_GIONEE_WBL7511
- https://github.com/SeriniTY320/android_device_THL4000-cm-13.0
- https://github.com/CyanogenMod/android_device_cyanogen_mt6735-common - android_device_cyanogen_mt6735-common
- https://github.com/Lucky76/android_device_ulefone_metal (MT6753, 64-bit)
- https://github.com/Mohancm/device_A7010a48 (MT6763, 64-bit)

В последнем дереве зачем-то сделан "интересный хак":

#service ril-daemon /system/bin/rild
service ril-daemon /system/bin/mtkrild

См. тут - https://github.com/Mohancm/device_A7010a48/blob/acbb4490d52f068f9950057a76cf936230544ca7/rootdir/etc/init.rc

Статьи:

- https://forum.xda-developers.com/k3-note/orig-development/rom-custom-nougat-roms-k-3-note-t3513466
- https://habrahabr.ru/post/183984/ (Слой радиоинтерфейса в ОС Android)

На тему решения проблемы с записью видео в камере:

- https://github.com/fire855/android_frameworks_av-mtk/commits/cm-13.0-mt6592
- https://github.com/nofearnohappy/device_hermes_cm13/blob/master/patches/frameworks/av/codec_and_audio.patch
- https://github.com/xen0n/android_device_meizu_arale/issues/17
- https://forum.xda-developers.com/k3-note/orig-development/rom-custom-nougat-roms-k-3-note-t3513466
- http://4pda.ru/forum/index.php?s=&showtopic=715583&view=findpost&p=57481976

Создал также отдельный репозитарий с описанием проблемы и полными логами:

https://github.com/DeckerSU/video_recording_problem_cm13/


[7] Уррра ... удалось завести запись видео с google.h264.encoder, однако пока что перепутаны цвета Red и Blue на
записи, т.е. красные объекты выглядят синими, а синие выглядят красными. Решаем эту проблему.

https://github.com/xen0n/android_device_meizu_arale/issues/17#issuecomment-274554382

***

Я пошел по другому пути. Т.е. есть два варианта как записывать видео:

1. Использовать софтовый google.h264.encoder из CM.
2. Использовать OMX.MTK.VIDEO.ENCODER.AVC.

Разбираться нужно последовательно, почему не работает в том и в другом случае. Я начал с варианта с google.h264.encoder. Медленно, но верно (весь сегодняшний вечер занимаюсь) я все-таки поднял (ура, ура, ура) съемку видео софтовым кодеком. Все снимает, ничего не крашится, но есть другая проблема. Color conversion, на записи перепутаны местами Red и Blue цвета. Т.е. например красный объект на записи будет выглядеть как синий, а синий, как красный )) Но эту проблему я знаю как решить, вернее знаю направление.

Когда решу ее, вторым шагом будет запуск OMX.MTK.VIDEO.ENCODER.AVC для кодирования видео. Я уже потихоньку начал понимать смысл вопроса. 

Вообщем сейчас у меня видео уже снимается, но с перепутанными цветами. Так что первый шаг сделан.

***

Это софтовый google.h264.encoder :

SoftOMXPlugin.cpp
    { "OMX.google.h264.encoder", "avcenc", "video_encoder.avc" },
libstagefright_soft_avcenc.so

А это хардверный MTK'шный:

01-24 02:33:46.132   845   955 D MtkOmxCore: name(OMX.MTK.VIDEO.DECODER.AVC), role(video_decoder.avc), path(libMtkOmxVdecEx.so)
01-24 02:33:46.132   845   955 D MtkOmxCore: name(OMX.MTK.VIDEO.DECODER.AVC.secure), role(video_decoder.avc), path(libMtkOmxVdecEx.so)

При записи видео, например, через screenrecord используется:

SoftVideoEncoderOMXComponent.cpp
SoftVideoEncoderOMXComponent::ConvertRGB32ToPlanar

И вот там как раз если включим:

#ifdef SURFACE_IS_BGR32
    bgr = !bgr;
#endif

То цвета в screenrecord'е на записи изменятся с RGB на BGR. Но (!) при использовании OMX.google.h264.encoder
при записи с камеры ConvertRGB32ToPlanar не вызывается (!), только при screenrecord'е. Запись же с камеры при
использовании OMX.google.h264.encoder сразу использует libstagefright_soft_avcenc.so. Т.е.

frameworks/av/media/libstagefright/codecs/avcenc/SoftAVCEnc.cpp .

Однако, при записи видео и кодировании google.h264.encoder (libstagefright_soft_avcenc.so) инициализируется
ColorConverter:

frameworks/av/media/libstagefright/colorconversion/ColorConverter.cpp 
ColorConverter::ColorConverter(
        OMX_COLOR_FORMATTYPE from, OMX_COLOR_FORMATTYPE to)
    : mSrcFormat(from),
      mDstFormat(to),
      mClip(NULL) {

ALOGE("[Decker] ColorConverter::ColorConverter: %#x --> %#x", from, to);

}

со следующими параметрами:

ColorConverter: [Decker] ColorConverter::ColorConverter: 0x13 --> 0x6
OMX_COLOR_FormatYUV420Planar -> OMX_COLOR_Format16bitRGB565

frameworks/native/include/media/openmax/OMX_IVCommon.h 

typedef enum OMX_COLOR_FORMATTYPE {
0    OMX_COLOR_FormatUnused,
1    OMX_COLOR_FormatMonochrome,
2    OMX_COLOR_Format8bitRGB332,
3    OMX_COLOR_Format12bitRGB444,
4    OMX_COLOR_Format16bitARGB4444,
5    OMX_COLOR_Format16bitARGB1555,
6    OMX_COLOR_Format16bitRGB565,
7    OMX_COLOR_Format16bitBGR565,
8    OMX_COLOR_Format18bitRGB666,
9    OMX_COLOR_Format18bitARGB1665,
0xa    OMX_COLOR_Format19bitARGB1666,
0xb    OMX_COLOR_Format24bitRGB888,
0xc    OMX_COLOR_Format24bitBGR888,
0xd    OMX_COLOR_Format24bitARGB1887,
0xe    OMX_COLOR_Format25bitARGB1888,
0xf    OMX_COLOR_Format32bitBGRA8888,
0x10    OMX_COLOR_Format32bitARGB8888,
0x11    OMX_COLOR_FormatYUV411Planar,
0x12    OMX_COLOR_FormatYUV411PackedPlanar,
0x13    OMX_COLOR_FormatYUV420Planar,
0x14    OMX_COLOR_FormatYUV420PackedPlanar,

ColorConverter: [Decker] ColorConverter::ColorConverter: 0x13 --> 0x6
OMX_COLOR_FormatYUV420Planar -> OMX_COLOR_Format16bitRGB565

Здесь вроде как OMX_COLOR_FormatYUV420Planar берется непосредственно с камеры, а вот target format =
OMX_COLOR_Format16bitRGB565 задается здесь:

frameworks/av/media/libstagefright/StagefrightMetadataRetriever.cpp 

ColorConverter converter((OMX_COLOR_FORMATTYPE)srcFormat, OMX_COLOR_Format16bitRGB565); (!)

Сама конвертация происходит в frameworks/av/media/libstagefright/colorconversion/ColorConverter.cpp 

        case OMX_COLOR_FormatYUV420Planar:
            err = convertYUV420Planar(src, dst);
* [State İdarəçiliyi](#state-management)
  + [Reactive State Manager](#reactive-state-manager)
    - [Üstünlüklər](#advantages)
    - [Maksimum performans](#maximum-performance)
    - [Reaktiv dəyişənin elan edilməsi](#declaring-a-reactive-variable)
        - [Reaktiv vəziyyətə sahib olmaq asandır.](#having-a-reactive-state-is-easy)
    - [Görünüşdəki dəyərlərdən istifadə](#using-the-values-in-the-view)
    - [Yenidən qurulma şərtləri](#conditions-to-rebuild)
    - [Harada .obs istifadə edilə bilər](#where-obs-can-be-used)
    - [Siyahılar haqqında qeyd](#note-about-lists)
    - [Nə üçün .value istifadə etməliyəm](#why-i-have-to-use-value)
    - [Obx()](#obx)
    - [İşçilər](#workers)
  + [Sadə State Meneceri](#simple-state-manager)
    - [Üstünlüklər](#advantages-1)
    - [İstifadəsi](#usage)
    - [Controller ləri necə idarə edir](#how-it-handles-controllers)
    - [Artıq StatefulWidget-lərə ehtiyacınız olmayacaq](#you-wont-need-statefulwidgets-anymore)
    - [Niyə var](#why-it-exists)
    - [Digər istifadə üsulları](#other-ways-of-using-it)
    - [Unikal ID-lər](#unique-ids)
  + [İki state idarəçisinin qarışdırılması](#mixing-the-two-state-managers)
  + [GetBuilder və GetX və Obx və MixinBuilder](#getbuilder-vs-getx-vs-obx-vs-mixinbuilder)
  + 
  + # State İdarəçiliyi

GetX digər State menecerləri kimi Streams və ya ChangeNotifier istifadə etmir. Niyə? GetX ilə siz Android, iOS, web, linux, macos və linux üçün proqramlar yaratmaqla yanaşı, Flutter/GetX ilə eyni sintaksislə server proqramları yarada bilərsiniz. Cavab müddətini və RAM istehlakını azaltmaq üçün biz daha az əməliyyat ilə daha çox performans təmin edə bilmə həlləri olan GetValue və GetStream yaratdıq. Biz bütün resurslarımızı, o cümlədən dövlət idarəçiliyini yaratmaq üçün bu təməldən istifadə edirik.

* _Mürəkkəblik_: Bəzi dövlət menecerləri mürəkkəbdirlər və çoxlu istəkləri var. GetX ilə siz hər bir hadisə üçün sinif təyin etməli deyilsiniz, kod çox təmiz və aydındır və siz daha az yazmaqla daha çox iş görürsünüz. Bir çox insanlar bu mövzuya görə Flutter-dən imtina etdirdilər yalnız indi nəhayət ki, state ləri idarə etmək üçün həddindən sadə bir həll yolu var.
* _Kod generatorları yoxdur_: Proqramı inkişaf etdirmə vaxrınızın yarısını məntiqləri axtarmaqla və onları yazmaqla keçirirsiniz. Bəzi state menecerləri minimum oxuna bilən koda sahib olmaq üçün kod generatorlarına güvənir. Dəyişənləri yeniləmək və build_runner-ı işə salmaq səmərəsiz ola bilir və tez-tez flutter clean dan sonra gözləmə müddəti uzun olduğu üçün siz boş keçəm zamanı çoxlu kofe içmək ilə keçirməli olacaqsınız.

GetX ilə hər şey reaktivdir və heç bir şey kod generatorlarından asılı deyil, inkişafın bütün aspektlərində məhsuldarlığınızı artırır.

* _Context dən asılı deyil_: Yəqin ki, siz artıq dəfələrlə kodlarınızda  context i istifadə etməli olmusunuz, təbii olaraq basqa class lara və functionlara context i ötürmıyin nə qədər vacib olduğunu görmüsünüz. Bu, sadəcə GetX-də lazım deyil. Controller lərinizə, controllerinizin içindən heç bir kontekst olmadan daxil ola bilərsiniz. Siz sözün əsl mənasında heç bir şey üçün conteksti parametrə olaraq göndərməyə ehtiyac duymayacaqsınız.
* _Granular control_: əksər state menecerlər ChangeNotifier-a əsaslanır. ChangeNotifier notifyListeners i çağırıldıqda ondan asılı olan bütün vidjetlər xəbərdar edilir. Bir ekranda ChangeNotifier sinifinizin dəyişəninə malik 40 varsa, birini yenilədiyiniz zaman onların hamısı yenidən qurulacaq.

GetX ile iç içə widget lara da fərdi yanaşıllır. ListView ınızı izləyən Obx iniz və ListView ın içində bir CheckBox izləyən başqa biri varsa, CheckBox dəyərini dəyişdirdiyi zaman yalnızca o yenilənəcəkdir, List dəyərini dəyişdirdiyiniz zaman isə yalnızca ListView yenilənəcəkdir.

* _Dəyişən HƏQİQƏTƏNDƏN dəyişdikdə o, yenidən qurulur_: GetX-də axın nəzarəti var, yəni 'Aygün' adlı bir data göstərsəniz, müşahidə olunan dəyişəni yenidən 'Aygün' olaraq dəyişdirsəniz, vidcet yenidən qurulmayacaq. Çünki GetX 'Aygün' datası artıq mətndə göstərildiyini bilir və lazımsız yeniləmə edib yüklənməni atyırmayacaq.

Mövcud state menecerlərinin əksəriyyəti (hamısı olmasa da) ekranda yenidən qurulacaq.

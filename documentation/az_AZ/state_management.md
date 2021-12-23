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

## Reactive State Meneceri

Reaktiv proqramlaşdırma bir çox insanı özündən uzaqlaşdıra bilər, çünki onun mürəkkəb olduğu deyilir. GetX reaktiv proqramlaşdırmanı olduqca sadə bir şeyə çevirir:

* StreamController yaratmağa ehtiyacınız olmayacaq.
* Hər dəyişən üçün StreamBuilder yaratmağa ehtiyacınız olmayacaq.
* Hər bir state üçün bir class yaratmağa ehtiyacınız olmayacaq.
* İlkin dəyər üçün get yaratmağa ehtiyacınız olmayacaq.

Get ilə reaktiv proqramlaşdırma setState istifadə etmək qədər asandır.

Təsəvvür edək ki, sizin ad dəyişəniniz var və onu hər dəfə dəyişdirdiyiniz zaman ondan istifadə edən bütün vidjetlərin avtomatik yenilənməsini istəyirsiniz.

Bu sizin say dəyişəninizdir:

``` dart
var name = 'Alakbar Heydarov';
```

Onu daima müşahidə etmək üçün sonuna ".obs" əlavə etmək kifayətdir:

``` dart
var name = 'Alakbar Heydarov'.obs;
```

Bu qədər sadə !

Bundan sonra biz bu reaktiv-".obs" dəyişənlərinə _Rx_ kimi istinad edə bilərik .   

Başlıq arxa planda nə etdik? Dəyişənin ilkin dəyərini “Alakbar Heydarov” olaraq təyin etdik, “Alakbar Heydarov” istifadə edən bütün vidcetlərə onların indi bu dəyişənə mənsub olduqlarını bildirdik və Rx dəyəri dəyişdikdə onlar aşağıdakı kimi dəyişməli olacaqlar.

Bu, Dartın imkanları sayəsində **GetX-in sehri** budur.

Ancaq bildiyimiz kimi, `Widget` yalnız funksiyanın daxilində olduqda dəyişdirilə bilər, çünki statik siniflərin "avtomatik dəyişmə" etməyə gücü yoxdur.

You will need to create a `StreamBuilder` , subscribe to this variable to listen for changes, and create a "cascade" of nested `StreamBuilder` if you want to change several variables in the same scope, right?

Siz `StreamBuilder` yaratmalı, dəyişikliklərə qulaq asmaq üçün bu dəyişənələrə abunə olmalı və eyni miqyasda bir neçə dəyişəni dəyişdirmək istəyirsinizsə, daxili StreamBuilder-in yaratmalısınız, elə deyilmi?

Yalnız yox, sizə `StreamBuilder` lazım deyil. Amma hansısa çağrılma metodu haqqında yanılmırsınız

Beləliklə, biz müəyyən bir Vidceti dəyişmək istəyəndə adətən çoxlu ümumi məlumatımız our, bu Flutter üsuludur. **GetX** ilə siz bu üzul kodları unuda bilərsiniz.



`StreamBuilder( … )` ? `initialValue: …` ? `builder: …` ? bunu unudun, sadəcə olaraq dəyişənin daxil olduğu Vidceti `Obx()` Vidcetinin içərisinə yerləşdirin.

``` dart
Obx (() => Text (controller.name));
```

_Yadda saxlamaq üçün nə lazımdır? _  Yalnız və yalnız `Obx(() =>` . 

`Obx` olduqca ağıllıdır və yalnız `controller.name` dəyəri dəyişdikdə dəyişəcək.

Əgər `name` : `"Alakbar"` dlrsa və siz onu yenidən `"Alakbar"` a ( `name.value = "Alakbar"` )  dəyişdirirsinizsə, bu, əvvəlki kimi eyni `dəyərdirsə`, ekranda heç nə dəyişməyəcək və `Obx` resurslara qənaət etmək üçün sadəcə etinasızlıq göstərəcək. yeni dəyər və Vidceti yenidən qurmayın. **Bu heyrətamiz deyilmi?**

> Bəs, `Obx` daxilində 5 _Rx_ dəyişənim varsa onda necə?

Onlardan hər hansı biri dəyişdikdə yalnız o zaman yeniləmə baş verəcək. 

> Bir sinifdə 30 dəyişənim varsa, birini yeniləyəndə o , həmin sinifdə olan bütün dəyişənləri yeniləyəcəkmi ?

Xeyr, sadəcə _Rx_ dəyişənini istifadə edən xüsusi Vidgetin yenilənəcəyinə arxayın ola bilərsiniz .

Beləliklə, **GetX** yalnız Rx dəyişəni dəyərini dəyişdikdə ekranı yeniləyir.

``` 

final isOpen = false.obs;

//Bu zaman hec bir dəyər dəyişməyəcək.
void onButtonTap() => isOpen.value=false;
```

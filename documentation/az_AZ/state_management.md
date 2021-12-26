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

Başlıq arxa planda nə etdik? Dəyişənin ilkin dəyərini “Alakbar Heydarov” olaraq təyin etdik, 
“Alakbar Heydarov” istifadə edən bütün vidcetlərə onların indi bu dəyişənə mənsub olduqlarını bildirdik və 
Rx dəyəri dəyişdikdə onlar aşağıdakı kimi dəyişməli olacaqlar.

Bu, Dartın imkanları sayəsində **GetX-in sehri** budur.

Ancaq bildiyimiz kimi, `Widget` yalnız funksiyanın daxilində olduqda dəyişdirilə bilər, çünki statik siniflərin "avtomatik dəyişmə" etməyə gücü yoxdur.

Siz `StreamBuilder` yaratmalı, dəyişikliklərə qulaq asmaq üçün bu dəyişənələrə abunə olmalı və eyni miqyasda bir neçə dəyişəni dəyişdirmək istəyirsinizsə, 
daxili StreamBuilder-in yaratmalısınız, elə deyilmi?

Yalnız yox, sizə `StreamBuilder` lazım deyil. Amma hansısa çağrılma metodu haqqında yanılmırsınız

Beləliklə, biz müəyyən bir Vidceti dəyişmək istəyəndə adətən çoxlu ümumi məlumatımız our, bu Flutter üsuludur. **GetX** ilə siz bu üzul kodları unuda bilərsiniz.



`StreamBuilder( … )` ? `initialValue: …` ? `builder: …` ? bunu unudun, sadəcə olaraq dəyişənin daxil olduğu Vidceti `Obx()` Vidcetinin içərisinə yerləşdirin.

``` dart
Obx (() => Text (controller.name));
```

_Yadda saxlamaq üçün nə lazımdır? _  Yalnız və yalnız `Obx(() =>` . 

`Obx` olduqca ağıllıdır və yalnız `controller.name` dəyəri dəyişdikdə dəyişəcək.

Əgər `name` : `"Alakbar"` dlrsa və siz onu yenidən `"Alakbar"` a ( `name.value = "Alakbar"` )  dəyişdirirsinizsə, bu, 
əvvəlki kimi eyni `dəyərdirsə`, ekranda heç nə dəyişməyəcək və `Obx` resurslara qənaət etmək üçün sadəcə etinasızlıq göstərəcək. 
Yeni dəyər və Vidceti yenidən qurmayın. **Bu heyrətamiz deyilmi?**

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

### Üstünlüklər

Güncəllənənlərə nəzarət etmək lazım olanda **GetX()** sizə kömək edir.

Əgər hər hansı bir fəaliyyət yerinə yetirərkən bütün dəyişənlər dəyişdirildiyi üçün `ukal id-lərə` ehtiyacınız yoxdursa, 
GetBuilder-dən istifadə edin, çünki bu Simple State Updater (setState() kimi) yalnız bir neçə kod sətirindən ibarətdir. 
Ən az CPU işlədilməsinə nail olmaq, yalnız bir məqsədi yerinə yetirmək (State-nin yenidən qurulması) və 
minimum mümkün resursları istehlak etmək üçün sadələşdirilmişdir.

**Güclü** State Manager ehtiyacınız varsa, **GetX** ilə düz yolda olduğunuza əmin ola bilməzsiniz.

Bu, dəyişənlərlə işləmir, hər şey __axıcılıq__ ilə gedir, daxildə olan hər şey hər şey `Streams`in altında baş verir.

Həmçinin siz _rxDart_ dan istifadə edə bilərsiniz, 
çünki hər şey `Streams`-dir, siz hər bir "_Rx_ dəyişəninin" addımlarını dinləyə bilərsiniz, 
içindəki hər şeyin `Streams` olması sizə bu sadəliyi yaradır.

Bu, sözün əsl mənasında BLoC yanaşmasıdır, MobX-dən daha asandır və kod generatorları və ya əlavə kod yazıları olmadan, 
siz sadəcə .obs ilə hər şeyi "Müşahidə olunana (Observable)" çevirə bilərsiniz.

### Maksimum performans:

Minimal yenidənqurma üçün ağıllı alqoritmə malik olmaqla yanaşı, 
**GetX** State-nin dəyişdiyinə əmin olmaq üçün müqayisə edənlərdən istifadə edir.

Tətbiqinizdə hər hansı bir səhvlə qarşılaşsanız və dublikat State dəyişikliyi təqdim etsəniz, 
**GetX** proqramın qəzaya uğramamasını təmin edəcək.

**GetX** ilə State yalnız `value` dəyişdikdə dəyişir.
**GetX** ilə MobX-dən istifadə arasındakı əsas fərq budur.
İki __müşahidə olunanı__ ( __observables__ ) birləşdirərkən, və birini dəyişən zaman, 
həmin müşahidə olunanın dinləyicisi də dəyişəcək.

**GetX**ilə isə iki dəyişəni birləşdirsəniz, `GetX()` (Observer() ilə oxşar) yalnız vəziyyətin real dəyişikliyini nəzərdə tutduqda  State yenidən quracaq.

### Reaktiv dəyişənin elan edilməsi

Dəyişənləri "observable"ə çevirməyin 3 yolu var.

1 - İlk istifadə **`Rx{Type}`**.

``` dart
// initial value is recommended, but not mandatory
final name = RxString('');
final isLogged = RxBool(false);
final count = RxInt(0);
final balance = RxDouble(0.0);
final items = RxList<String>([]);
final myMap = RxMap<String, int>({});
```

2 - İkincisi, **`Rx`** istifadə etmək və Darts Generics istifadə etməkdir. `Rx<Type>`

``` dart
final name = Rx<String>('');
final isLogged = Rx<Bool>(false);
final count = Rx<Int>(0);
final balance = Rx<Double>(0.0);
final number = Rx<Num>(0);
final items = Rx<List<String>>([]);
final myMap = Rx<Map<String, int>>({});

// Custom classes - it can be any class, literally
final user = Rx<User>();
```

3 - Üçüncü, daha praktik, asan və üstünlük verilən yanaşma, sadəcə olaraq `dəyərinizin` sonuna **`.obs`** əlavə edin:

``` dart
final name = ''.obs;
final isLogged = false.obs;
final count = 0.obs;
final balance = 0.0.obs;
final number = 0.obs;
final items = <String>[].obs;
final myMap = <String, int>{}.obs;

// Custom classes - it can be any class, literally
final user = User().obs;
```

##### Reaktiv vəziyyətə sahib olmaq asandır.

Bildiyimiz kimi, _Dart_ indi s_null safety_ yə doğru gedir. 
Heç bir problemin yaşanmaması üçün bundan sonra həmişə _Rx_ dəyişənlərinizi **ilkin dəyərlə** başlamalısınız.

> Bir dəyişəni _observable_ çevirmək + _ilkin dəyər_ vermək **GetX** də sadə və çox rahatdır.

Dəyişəninizin sonuna  `.obs` əlavə edəcəksiniz və bu, siz onu müşahidə edilə bilən ( observable ) etdiniz 
və onun istifadə edən zaman `.value` onun _ilkin dəyər_ olacaq.

### Görünüşdəki dəyərlərdən istifadə

``` dart
// controller file
final count1 = 0.obs;
final count2 = 0.obs;
int get sum => count1.value + count2.value;
```

``` dart
// view file
GetX<Controller>(
  builder: (controller) {
    print("count 1 rebuild");
    return Text('${controller.count1.value}');
  },
),
GetX<Controller>(
  builder: (controller) {
    print("count 2 rebuild");
    return Text('${controller.count2.value}');
  },
),
GetX<Controller>(
  builder: (controller) {
    print("count 3 rebuild");
    return Text('${controller.sum}');
  },
),
```

Əgər `count1.value++` ilə artırsaq, çap olacaq:

* `count 1 rebuild`

* `count 3 rebuild`

çünki `count1` in  `1` dəyəri var, və `1 + 0 = 1` , `sum` öz dəyərini bu çür dəyişəcəkdir.

Əgər `count2.value++` ni dəyişsək , çap olacaq:

* `count 2 rebuild`

* `count 3 rebuild`

Çünki `count2.value` dəyişdi, və nətiçə olaraq `sum` daha `2` dir.

* QEYD: Varsayılan olaraq, eyni dəyər olsa belə, ilk hadisə widget i yenidən quracaq.

 Bu davranış Boolean dəyişənlərinə görə mövcuddur.

Təsəvvür edin ki, siz bunu etdiniz:

``` dart
var isLogged = false.obs;
```

Və sonra, istifadəçinin "logged in" əmrinə uyğun olub oladığını yoxladınız _ever_.
.

``` dart
@override
onInit() async {
  ever(isLogged, fireRoute);
  isLogged.value = await Preferences.hasToken();
}

fireRoute(logged) {
  if (logged) {
   Get.off(Home());
  } else {
   Get.off(Login());
  }
}
```

əgər `hasToken` `false` olsaydı, `isLogged`-də heç bir dəyişiklik olmazdı, ona görə də `ever()` heç vaxt çağırılmazdı. 
Bu cür davranışın qarşısını almaq üçün _observable_ də ilk dəyişiklik həmişə eyni.
hətta eyni `.value` ehtiva etsə də .

İstəsəniz, bu davranışı aradan qaldıra bilərsiniz:
 `isLogged.firstRebuild = false;`

### Yenidən qurmaq üçün şərtlər


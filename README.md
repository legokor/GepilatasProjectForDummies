# Bevezetés a gépi látásba
## Bevezető
A számítógépes képfeldolgozás során először a számunkra érdekes objektum körvonalait próbáljuk beazonosítani. Ezután a színe, pontos körvonala alapján dolgozzuk fel (pl.: SET kártya párosító: először kártyák beazonosítása körvonallal, és azon belül az alakzatok megkeresése ugyancsak körvonalakkal).
Ahhoz, hogy megtaláljuk a körvonalat a keresés előtt néhány előfeldolgozó műveletet érdemes végezni (pl.: szín szűrés, elmosás, erózió), ezekről később részletesebben lesz szó.
A feldolgozás után érdemes valamilyen visszajelzést adni a felhasználónak, hogy a program mit lát, ezért általában az eredeti képre végzett rajzoló műveletek után ki is rajzoljuk a képet.

![Image processing pipeline](/pictures/pipeline.jpg "A képfeldolgozás általános folyamata (nem mindegi szükséges az összes elem)")

A projektben legtöbben OpenCV könyvtárat használunk, mivel ez sok nyelven elérhető, ingyenes és kellően funkciógazdag. Persze nem kötelező OpenCV-t használni a projektjeidhez :). <br> A kódreszletek C++ nyelven íródtak, mivel ez az OpenCV könyvtár eredeti nyelve, ez alapján más nyelveken is egyszerűen kitalálható a működés és remélhetőleg már ti is tanultatok (Prog2-ből).

## Színterek
A különböző színterek más-más logika mentén ábrázolják a színeket, így különböző műveletek különböző színterekben végezhetőek el könnyedén.

### BGR
Lényegében ugyan az, mint a közismert RGB, csak más sorrendben vannak a csatornák. *Alapértelmezetten ilyen formában olvassa be az OpenCV a képet.* A színeket 0-255-ig terjedő skálán kék (**B**lue), zöld (**G**reen) és piros (**R**ed)csatornákra bontva ábrázolja. Egyes esetekben egy A csatorna is tartozhat hozzá, ami az átlátszóságot jelenti.
![BGR Color space](/pictures/colorspace/RGB_colorspace.jpg "A BGR színtér")

### HSV
A HSV (**H**ue, **S**aturation, **V**alue) színtér egy henger-koordináta rendszerrel ábrázolja a színeket. 

- A **H** a középponti szög. A hengeren körben pirostól indulva, az összes színen át ismét pirosba érve vannak a színek elhelyezve, ezek közül jelöl ki egyet.(elvileg 0-360°-ig, OpenCV-ben 0-179)

![Color spectrum](/pictures/colorspace/colorspectrum.jpg "A színek 0-360°-ig")

- Az **S** a sugár, gyakorlatilag azt mutatja meg mennyire élénk a szín. (0-255)
- A **V** paraméter a magasság és a szín világosságát mutatja. (0-255)

![HSV Color space](/pictures/colorspace/HSV_colorspace.jpg "A HSV színtér")

Mivel ez a színtér egy értéken ábrázolja azt, hogy az adott pixel ténylegesen milyen színű (zöld, narancssárga), ezért ebben a színtérben rendkívül egyszerű a szín szűrés, átszínezés.

### LAB
A LAB (**L**ightness , **a**, **b**) színtér is 3 komponensből áll, viszont már egy fokkal elvontabb.

- Az **L** a világosságot jelöli
- Az **a** a színt zöldtől magentáig
- A **b** a színt kéktől sárgáig

Ez leginkább fotószerkesztésből lehet ismerős, a fehér egyensúlyt lehet ilyen skálán állítani. Előnye, hogy készülékfüggetlen (mindegy milyen érzékelővel érzékeltük a fényt és milyen kijelzőn jelenítjük meg, ugyan úgy néz ki).

![LAB Color space](/pictures/colorspace/LAB_colorspace.gif "A CIELAB színtér")

### YCrCb

- Az **Y** (aposztróf nélkül, létezik Y'CrCb tér is) a fényességet jelzi (Luma, luminance). Ezt RGB-ből, gamma-korrekcóval állítják elő
- A **Cr** a piros Luma-tól való távolságát adja meg (Cr=R-Y)
- A **Cb** a kék Luma-tól való távolságát adja meg (Cr=B-Y)

TV adásnál és a JPEG formátumnál alkalmazzák tömörítéshez. Készülékfüggő.

![YCrCb Color space](/pictures/colorspace/YCbCr_colorspace.gif "Az YCrCb színtér")

### CMYK
A nyomdában használt cián (**c**yan), **m**agenta, citromsárga (**y**ellow), és fekete (blac**k**) színekre bontva írja le az adott színt. Mi nem igazán használjuk, de a teljesség kedvéért megemlítettem.

[Rubik kocka esettanulmány](https://www.learnopencv.com/color-spaces-in-opencv-cpp-python/)

---

## OpenCV obejktumok, függvények
Az OpenCV az egyes fügvényeket illetve osztályokat funkció szerint külön modulokban tárolja. Ebből egészen sok van, funkciójuk a képfeldolgozás, képmanipulálástól kezdve a mély neurális hálókig és gépi tanulásig egyéb bonyolultabb dolgokig terjed. Az alábbiakban az (általunk) leggyakrabban használt, alap modulok függvényeiről, osztályairól lesz szó:
- **core:** alapvető struktúrák és osztályok
- **imgcodecs:** a különböző formátumok beolvasásához, illetve a nagyon egyszerű előfeldolgozó műveletekhez tartozó függvények (pl.: két kép összeadása)
- **imgproc:** képfeldolgozó műveletek
- **highgui:** magas szintű GUI eszközök (pl.: Trackbar, esemény kezelés)

[További modulok](https://docs.opencv.org/3.4/modules.html)

### Alapvető tárolók

#### Point
Egy 2D pontot reprezentál, `x` és `y` koordinátákkal. Az alábbi módokon definiálható:
```c++
Point pt;
pt.x = 10;
pt.z = 8;
```
```c++
Point pt = Point(10, 8)
```
A Point [ennek a template-nek](https://docs.opencv.org/3.4/db/d4e/classcv_1_1Point__.html) \<int\> paraméterrel typedefelve.

#### Scalar
Egy 4 elemű vektort reprezentál (és igen, skalár a neve...), de ennél kevesebbel is példányosítható. Gyakran használjuk a pixel értékek átadására.
```c++
Scalar(5, 6, 7);
```

A Scalar [ennek a template-nek](https://docs.opencv.org/3.4/d1/da0/classcv_1_1Scalar__.html) \<double\> paraméterrel typedefelve.

#### MAT
[core modul](https://docs.opencv.org/3.4/da/d47/core_8hpp.html)
A számítógép a képeket mátrixokban tárolja ("olyan táblázat amiben számok vannak"). Minden pixelhez tartozik egy cella. Ez szürkeárnyalatos képek esetén 1 számot (ez az intenzitás), a fentebb leírt színábrázolási módok esetén 3-4 számot tartalmaz.
A `Mat` objektum két részből áll: 
- egy fejlécből, ami meta infokat táról a mátrixról (méret, tárolási mód, mátrix címe), ennek a mérete viszonylag kicsi és fix
- magából a képet leíró mátrixból

Ez azért hasznos, mert így több fejléc is utalhat ugyanarra a mátrixra, akár különböző részeire (ROI = region of interest), így kevesebb memóriát használ.
![Multiple headers](/pictures/multiheader.jpg "Két header mutathat ugyanarra a mátrixra, vagy egy részére")

A másoló operátorok ezért csaka fejlécet másolják le ténylegesen, magát a nagy mátrixot nem.
```c++
//Eredeti mátrix
Mat A = cv::imread("image.jpg");
//Copy konstruktor
Mat B(A);
//Értékadás operátor
Mat C;
C = A;
```
Ebben az esetben csak a fejléc másolódik le, ugyanarra a memória területre fog hivatkozni mind a 3, tehát az az egyiket módosítjuk, akkor a másik 2 is módosulni fog.

Ahhoz, hogy ténylegesen lemásoljuk a mátrixot az alábbi műveletek egyike fog kelleni:
```c++
Mat cv::Mat::clone() const;
```
- **return**: egy teljesen új másolatot készít a mátrixról, és ezt adja vissza
[További info](https://docs.opencv.org/3.4/d3/d63/classcv_1_1Mat.html#adff2ea98da45eae0833e73582dd4a660)
```c++
void cv::Mat::copyTo(OutputArray m) const;
```
- **m**: cél objektum ahova a másolatot létrehozza. Ha volt benne már mátrix, akkor azt újra allokálja. *

Két paraméteres változat:
```c++
void cv::Mat::copyTo(OutputArray m, InputArray mask) const;
```
- **m**: ugyan az
- **mask**: CV_8U, egy vagy többcsatornás kép, **m**-el megegysező méretű. A másolás csak ennek a nem nulla elemeinél történik meg m-be

[További info](https://docs.opencv.org/3.4/d3/d63/classcv_1_1Mat.html#a33fd5d125b4c302b0c9aa86980791a77)

```c++
//Eredeti mátrix
Mat A = cv::imread("image.jpg");
//Klónozás
Mat F = A.clone();
//Copy hívás
Mat G;
A.copyTo(G);
```
Ebben az esetben a memóriában másik területre mutat mind a 3 fejléc, ha az egyiket módosítjuk, az a többin nem változtat.

Ha a mátrixnak csak egy részére van szükség (pl.: azt kell részletesebben elemezni), akkor az alábbi módon lehet kiválasztani:
```c++
//Téglalappal
Mat D (A, Rect(10, 10, 100, 100) );
```
[Rect](https://docs.opencv.org/3.4/dc/d84/group__core__basic.html#ga11d95de507098e90bad732b9345402e8)

Azt, hogy ne nekünk kelljen észben tartani, hogy éppen hány fejléc mutat a mátrxira, az OpenCV referencia számlálással megoldja. Csak akkor törli ki magát a mátrixot, ha a mátrixra mutató fejlécek száma eléri a nullát.
Mivel általában nem kell expliciten létrehoznunk mátrixot, ezért a `cv::Mat::Mat()` konstruktort valamint a `cv::Mat::create()` függvényt és végtelen overload-olt változatukat nem részletezném, további info mellett itt megtalálhatjátok:
[Learn more](https://docs.opencv.org/3.4/d6/d6d/tutorial_mat_the_basic_image_container.html)

A mátrix egyes elemeit a
```c++
_Tp& cv::Mat::at(int row, int col);
```
függvénnyel lehet elérni. Ez egy referenciát ad vissza a mátrix megfelelő sorában és oszlopában elhelyezkedő elemre. Ahhoz, hogy helyesen értelmezzük a lekért adatot, tudnunk kell, hogy milyen színtérben van a kép eltárolva.
```c++
//Beolvassuk a képet
Mat img = cv::imread("image.jpg", IMREAD_COLOR);
//Átkonvertáljuk szürkeárnyalatossá
Mat img_gray;
cv::cvtColor(img, img_gray, COLOR_BGR2GRAY);
//A szürkeárnyalatos képen 1 csatorna van, tehát skalárt kapunk vissza
//Mivel alapértelmezetten 8 biten tárolódik az intenzitás, unsigned char-t várunk
//Szokás szerint a mátrixnak először a sorát, utána az oszlopát kell címezni
Scalar intensity = img.at<uchar>(y, x);
//C++-ban használható még ez a formája is:
intensity = img.at<uchar>(Point(x, y));
//Ebben az objektumban a 0-s indexre került a tényleges 0-255 közötti érték
uchar actualIntensity = intensity[0];
```
[Scalar](https://docs.opencv.org/3.4/dc/d84/group__core__basic.html#ga599fe92e910c027be274233eccad7beb), [Point](https://docs.opencv.org/3.4/dc/d84/group__core__basic.html#ga1e83eafb2d26b3c93f09e8338bcab192), [uchar](https://docs.opencv.org/3.4/d1/d1b/group__core__hal__interface.html#ga65f85814a8290f9797005d3b28e7e5fc)
```c++
//Beolvassuk a képet
Mat img = cv::imread("image.jpg", IMREAD_COLOR);
//A BGR kép 3 csatornás, tehát egy 3 komponensű vektort kapunk vissza
Vec3b intensity = img.at<Vec3b>(y, x);
uchar blue = intensity.val[0];
uchar green = intensity.val[1];
uchar red = intensity.val[2];
```
[Vec3b](https://docs.opencv.org/3.4/dc/d84/)

[Learn more](https://docs.opencv.org/3.4/d5/d98/tutorial_mat_operations.html)

A méretet az `int cv::Mat::rows` és az `int cv::Mat::cols` adattaggal lehet elérni. Ha több mint 2 dimenziójú a mátrix, akkor a `MatSize cv::Mat::size`-t éredemes használni.
[Még több tagfüggvény és adattag itt](https://docs.opencv.org/3.4/d3/d63/classcv_1_1Mat.html)
> más típusok?
### Kép beolvasása
[imgcodecs modul](https://docs.opencv.org/3.4/d4/da8/group__imgcodecs.html)
Képet beolvasására használható függvény:
```c++
Mat cv::imread(const String& filename, int flags=IMREAD_COLOR);
```
- **filename**: a file relatív, vagy abszolút elérési útvonala
- **flags**: milyen típusú képként olvassa be a képet. A gyakoriakat alább találjátok, [a teljes listát pedig itt](https://docs.opencv.org/3.4/d4/da8/group__imgcodecs.html#gga61d9b0126a3e57d9277ac48327799c80af660544735200cbe942eea09232eb822)
  - `IMREAD_COLOR`: 3 csatornás BGR formátumban olvassa be a képet
  - `IMREAD_GRAYSCALE`: 1 csatornás szürkeárnyalatos képként olvassa be

A JPG, JPEG és BMP formátumok mellett sok másikat is támogat.
[További info](https://docs.opencv.org/3.4/d4/da8/group__imgcodecs.html#ga288b8b3da0892bd651fce07b3bbd3a56)

```c++
Mat img = cv::imread("image.jpg", IMREAD_COLOR);
```
### Kép kirajzolása
[highgui module](https://docs.opencv.org/3.4/d4/dd5/highgui_8hpp.html)
Ahhoz, hogy egy képet ki tudjunk rajzolni először szükségünk van egy ablakra. Ezt az alábbi függvénnyel tudjuk létrehozni:
```c++
void cv::namedWindow(const String& winname, int flags = WINDOW_AUTOSIZE);
```
- **winname**: az ablak neve, ezzel lehet rá később hivatkozni
- **flags**: egyéb opciók, pl.: változtatható ablakméret, aránytartás, eszköz sáv megjelnítése. Ezek egy része (pl.: változtatható ablak méret) Qt-vel használhatók (GUI környezet C++-hoz), ezért ezeket nem részletezem. 

[További info](https://docs.opencv.org/3.4/d7/dfc/group__highgui.html#ga5afdf8410934fd099df85c75b2e0888b)

Alapértelmezetten az ablak pont akkora lesz, hogy a kép (és esetleg plusz GUI elemek elférjenek rajta). A képet ezzel a függvénnyel tudjuk megjeleníteni:
```c++
void cv::imshow(const String& winname, InputArray mat);
```
- **winname**: az ablak neve, amin megjelenjen a kép
- **mat**: a megjelenítendő kép (tipikusan: Mat)

[További info](https://docs.opencv.org/3.4/d7/dfc/group__highgui.html#ga453d42fe4cb60e5723281a89973ee563)

Ahhoz, hogy a kép ne tűnjön el rögtön, érdemes a megjelenítés után tenni egy gombra várakozást:
```c++
int cv::waitKey(int delay = 0);
```
- **delay**: a függvény legalább ennyi milliszekundumig várakozik az eseményre (szálkezelés miatt nem pontos). A 0 speciális érték: ekkor végtelen ideig vár

- **return**: a megnyomott billentyű kódja, vagy -1, ha nem történt lenyomás

[További info](https://docs.opencv.org/3.4/d7/dfc/group__highgui.html#ga5628525ad33f52eab17feebcfba38bd7)

```c++
//Kép beolvasása
Mat img = cv::imread("image.jpg");
//Ablak létrehozása
cv::namedWindow("Image viewing example");
//Kép kirajzolása
cv::imshow("Image viewing example", img);
//Várakozás billentyű lenyomásra
cv::waitKey();
```
[Learn more](https://docs.opencv.org/3.4/db/deb/tutorial_display_image.html)
### Kép mentése
[imgcodecs modul](https://docs.opencv.org/3.4/d4/da8/group__imgcodecs.html)
A képek fájlbairására használható függvény a
```c++
bool cv::imwrite  (const String&  filename, InputArray img, const std::vector<int>&  params = std::vector< int >()); 
```
- **filename**: a file neve (elérési úttal együtt, ha máshová akarjuk menteni), amibe menteni szeretnénk. A formátumot ez alapján határozza meg az OpenCV.
- **img**: a kép amit menteni szeretnénk. Alapvetően BGR formátumba kell konvertálni a képet mentés előtt.
- **params**: formátum specifikus paraméterek, páronként kódolva. Erről részletek [itt találhatóak](https://docs.opencv.org/3.4/d4/da8/group__imgcodecs.html#ga292d81be8d76901bff7988d18d2b42ac)

[További info](https://docs.opencv.org/3.4/d4/da8/group__imgcodecs.html#gabbc7ef1aa2edfaa87772f1202d67e0ce)

```c++
//Kép beolvasása
Mat img = cv::imread("image.jpg");
//Szemléltetésként szürkeárnyalatossá változtatjuk a képet
//Mat object a szürkeárnyalatos képhez:
Mat gray_img;
//Konvertálás szürkeárnyalatossá: (később részletezve lesz)
cv::cvtColor(img, gray_img, COLOR_BGR2GRAY);
//Fájlba mentés
cv::imwrite("gray.jpg", gray_img);
```

[Learn more](https://docs.opencv.org/3.4/db/d64/tutorial_load_save_image.html)

### Képek összeadása
[modul]
Két kép összeadására nem csak áttűnéses-felemás képek készítése esetén van szükség. Több szűrés eredményét például így  tudjuk egy képre összegezni (pl.: a piros színre szűrésnél a két tartományt összevonni). Erre ad lehetőséget a `cv::addWeighted` függvény, amiben súlyt is rendelhetünk a tagokhoz. Az eredmény kiszámolása pixelenként az alábbi módon történik:
`dst = src1 * alpha + src2 * beta + gamma`
```c++
void cv::addWeighted  (InputArray src1, double alpha, InputArray src2, double beta, double  gamma, OutputArray dst, int dtype = -1); 
```
- **src1**: az egyik kép amit szeretnénk összeadni (tpikusan: Mat)
- **alpha**: az első kép súlya
- **src2**: a másik kép amit szeretnénk összeadni (tipikusan: Mat)
- **beta**: a második kép súlya
- **gamma**: az összeghez adott skalár
- **dst**: cél mátrix, ugyanakkor mérettel és dimenzióval, mint a 2 bemenet (a két bemenetnek egyező méretűnek kell lennie)
- **dtype**: a kimenet mélysége, ha a két bemenetre megegyezik, akkor -1-re állítható

[További info](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#gafafb2513349db3bcff51f54ee5592a19)

```c++
//Mátrixok létrehozása
Mat src1, src2, dst;
//Képek beolvasása
src1 = cv::imread("image1.jpg");
src2 = cv::imread("image2.jpg");
//Képek összeadása
cv::addWeighted(src1, 0.5, src2, 0.5, 0.0, dst);
//kiejelezzük
cv::imshow("Weighted sum", dst);
cv::waitKey(0);
```
>Kép beszúrása az eredményről

[Learn more](https://docs.opencv.org/3.4/d5/dc4/tutorial_adding_images.html)

### Maszkolt műveletek
A komplikáltabb szűrő, vagy előfeldolgozó algoritmusok egy maszkot használnak a művelet elvégzéséhez, ezt kernel-nek is nevezzük. A kernel az egy kisebb mátrix, ami súlyokat tartalmaz. A művelet során minden képpontra ráhelyezzük a kernel középső celláját, és a képpont környezetét a kernel mátrixban meghatározott súlyuk szerint összegezzük.
>Valami szemléltető kép

Mivel ez gyakran használatos, az OpenCV biztosít optimalizált függvényt rá, hogy ne nekünk kelljen kézzel megírni:
```c++
void filter2D(InputArray src, OutputArray dst, int ddepth, InputArray kernel, Point anchor=Point(-1,-1), double delta=0, int borderType=BORDER_DEFAULT);
```
- **src**: forrás kép (tipikusan: Mat)
- **dst**: cél kép, ugyanakkorának kell lennie méretre és csatorna számra, mint a forrásnak (tipikusan: Mat)
- **ddepth**: a kívánt mélység, negatív szám esetén megegyezik a forráséval (további info a linken)
- **kernel**: a kernelként használt mátrix, 1 csatornás (ha a különböző szín csatornákhoz különbözőt szeretnél használni: [split()](https://docs.opencv.org/2.4/modules/core/doc/operations_on_arrays.html))
- **anchor**: a kernel melyik celláját tekintse középpontnak (amelyiket ráilleszti a pixelekre), (-1;-1) esetén a kernel közepe
- **delta**: a súlyozott összegzés után hozzáadható skalár a pixelekhez
- **borderType**: pixel extrapolációs metódus ([borderInterpolate()](https://docs.opencv.org/2.4/modules/imgproc/doc/filtering.html?highlight=filter2d))

[Tovább info](https://docs.opencv.org/2.4/modules/imgproc/doc/filtering.html?highlight=filter2d#filter2d)

```c++
//Kép beolvasása
Mat img = cv::imread("image.jpg");
//Cél hely előkészítése
Mat dst;
//Kernel létrehozása
Mat kernel = (Mat_<char>(3,3) <<  0, -1,  0,
                                 -1,  5, -1,
                                  0, -1,  0);
//Filter alkalmazása
cv::filter2d(src, dst, src.depth(), kernel);
//Megjelenítés
cv::imshow("Filter result", dst);
cv::waitKey(0);
```
>Kép beszúrása az eredményről

Learn more [here](https://docs.opencv.org/3.4/d7/d37/tutorial_mat_mask_operations.html) and [here](https://docs.opencv.org/2.4/doc/tutorials/imgproc/imgtrans/filter_2d/filter_2d.html)

### Konverzió színterek között
A színterek közötti konverzióra a
```c++
void cv::cvtColor(InputArray src, OutputArray dst, int code);
```
függvény szolgál.

- **src**:forrás kép (a Mat objektum)
- **dst**: cél (Mat) objektum
- **code**: megmutatja miről mire akarunk változtatni


**TODO: bele kerül-e a dst-be a kép akármi van? pl.: már van benne másik mátrix, nem egyezik a méret**

#### Művelet kód
Bármely 2 színtér közötti váltáshoz definiálva van egy integer művelet kód. Ennek a konvenciója: `cv::COLOR_<MIRŐL>2<MIRE>`.
pl.:`COLOR_BGR2HSV`,`COLOR_BGR2GRAY`

```c++
//Kép beolvasása
Mat img = cv::imread("image.jpg");
//Transzformálás HSV tartományba
Mat hsv;
cv::cvtColor(img, hsv, cv::COLOR_BGR2HSV);
//Eredmény kirajzolása
cv::imshow("HSV color space", hsv);
cv::waitKey(0);
```
>Kép az eredményről

Az eredmény azért néz ki ilyen furcsán, mert az imshow() az BGR-ként értelmezi a kép 3 komponensét.

Amennyiben az új színtér tartalmaz alpha értéket (átlátszóság), akkor az maximális lesz (tehát egyáltalán nem fog átlátszani).

Az élkereső és egyéb algoritmusok alapvetően szürkeárnyalatos képen futnak, ezért előbb-utóbb mindenképpen át kell ilyenné konvertálni a képet, ha eddig nem történt meg (pl.:`COLOR_BGR2GRAY`).

## Előfeldolgozó műveletek
[imgproc modul](https://docs.opencv.org/3.4/d7/da8/tutorial_table_of_content_imgproc.html)
Az előfeldolgozás során próbálunk a saját és az élkereső algoritmus keze alá dolgozni. Sokkal könnyebb például egy piros labdát megkeresni a képen, ha már csak a piros dolgok körvonalát kapjuk meg, nem pedig az összes körvonalat a képen.
### Threshold
#### Egyszerű
Szürkeárnyalatos (1 csatornás) képekre lehet alkalmazni. Az átadott értékkel összehasonlítja a kép pixeleit és a típusától függően egy értéket rendel hozzá (pl.: ha kiseb feketét, ha nagyobb fehéret). Így ki lehet választani azt a részét a képnek, ami számunkra érdekes, vagy egy durva zaj szűrésre is jó.
```c++
double cv::threshold(InputArray src, OutputArray dst, double thresh, double maxval, int type);
```
- **src**: a forrás kép, 1 csatornás (tipikusan: Mat)
- **dst**: a cél kép, ahova az eredményt írja (tipikusan: Mat)
- **thresh**: az érték amivel összehasonlítja a pixelek értékét
- **maxval**: néhány típusnál a hozzárendelendő értéket ez adja meg
- **type**: a threshold típusa, előre definiáltak
  - `cv.THRESH_BINARY`: ha a pixel értéke nagyobb az összehasonlítási értéknél, akkor `maxval`-t, egyébként 0-t rendel hozzá
  - `cv.THRESH_BINARY_INV`: az előző fordítottja, ha a pixel értéke nagyobb az összehasonlítási értéknél, akko 0-t, egyébként `maxval`-t rendel hozzá
  - `cv.THRESH_TRUNC`: ha pixel értéke meghaladja az összehasonlítási értéket, akkor az összehasonlítási értéket rendeli hozzá, egyébként pedig az eredeti értéket
  - `cv.THRESH_TOZERO`: az összehasonlítási érték felett meghadja a pixel értékét, egyébként pedig 0-ra csökkenti
  - `cv.THRESH_TOZERO_INV`: az összehasonlítási érték alatt meghadja a pixel értékét, egyébként pedig 0-ra csökkenti

[Szemléltető ábrák](https://docs.opencv.org/3.4/db/d8e/tutorial_threshold.html)

```c++
//Kép beolvasása
Mat img = cv::imread("image.jpg");
//Transzformálás szürkeárnyalatossál
Mat gray;
cv::cvtColor(img, gray, cv::COLOR_BGR2GRAY);
//Threshold alkalmazása
Mat thresh;
cv::threshold(gray, thresh, 150, 255, cv.THRESH_BINARY);
//Eredmény kirajzolása
cv::imshow("Threshold", thresh); //TODO: imshow rendesen legyen megírva
cv::waitKey(0);
```
>kép az eredményről

#### Adaptív
Az előző függvény egy fix értékkel dolgozott az egész képen. Ez a függvény minden pixelhez külön határozza meg az összehasonlítási értéket. Ez is csak 1 csatornás képekkel működik.
```c++
void adaptiveThreshold(InputArray src, OutputArray dst, double maxValue, int adaptiveMethod, int thresholdType, int blockSize, double C);
```
- **src**: a forrás kép, *8 bites*, 1 csatornás (tipikusan: Mat)
- **dst**: a cél kép, ahova az eredményt írja (tipikusan: Mat)
- **maxValue**: a hozzárendelendő értéket ez adja meg
- **adaptiveMethod**: milyen módon számolja ki az összehasonlítási értéket a függvény (előre definiáltak)
  - `ADAPTIVE_THRESH_MEAN_C`: az összehasonlítási érték képzéshez az adott pixel `blockSize*blockSize` méretű környezetében átlagolja a pixelek értékét, és kivonja belőlük `C`-t
  - `ADAPTIVE_THRESH_GAUSSIAN_C`: az előzőhöz hasonló, viszont itt a súlyozott átlagát veszi a `blockSize*blockSize` méretű környezetnek (Gauss-ablak szerint), és ebből vonja ki a `C`-t
- **thresholdType**: a threshold típusa
  - `cv.THRESH_BINARY`: ugyan az, mint a sima thresholdnál, viszont itt az összehasonlítási értéket az előző módszerek szerint számolja ki, nem mi adjuk meg
  - `cv.THRESH_BINARY_INV`: ugyan az, mint a sima thresholdnál, viszont itt az összehasonlítási értéket az előző módszerek szerint számolja ki, nem mi adjuk meg
- **blockSize**: a pixel mekkora környezetét vegye figyelembe az átlagolásnál, csak páratlan szám lehet (a negyzet közepe lesz az aktuális pixel)
- **C**: az átlagból kivonandó konstans, általában pozitív, de lehet 0 vagy negatív is
```c++
TODO: PÉLDA KÓD
```
>kép az eredményről

[Learn more](https://docs.opencv.org/3.4.3/d7/d4d/tutorial_py_thresholding.html)
### Szín szűrés
Mint már említettem a színszűrést HSV színtérben szoktuk elvégezni. Ehhez a konverzió után az alábbi függvény használható:
```c++
void cv::inRange(InputArray src, InputArray lowerb, InputArray upperb, InputArray dst);
```
- **src**: a kép amit szűrni szeretnénk (tipikusan: Mat)
- **lowerb**: a szűrő alsó határa (tipikusan: Scalar), *ügyeljünk rá, hogy annyi elemszáma legyen, mint a kép egy pixelének (pl.: RGB: 3, GRAY: 1 - de itt is tömb)*
- **upperb**: a szűrő felső határa (tipikusan: Scalar), *ügyeljünk rá, hogy annyi elemszáma legyen, mint a kép egy pixelének (pl.: RGB: 3, GRAY: 1 - de itt is tömb)*
- **dst**: az objektum ahova menteni szeretnénk a kimenetet

A függvény lényegében minden pixelre leellenőrzi, hogy a pixel összes komponense benne van-e a kijelölt tartományban:
```
FOR pixelek IN image
    HA lowerb[0] <= pixel[0] <= upperb[0] AND lowerb[1] <= pixel[1] <= upperb[1] AND ...
        out.pixel = 1
    EGYÉBKÉNT
        out.pixel = 0
```
*A kimenet az egy bináris kép (csak fekete vagy fehér színekből áll, nincs átmenet).* Ha a pixel beleesik a tartományba, akkor fehér, egyébként fekete. Piros színre szűrés esetén érdemes figyelni arra, hogy a H tartomány elején és végén is van.
```c++
//Kép beolvasása
Mat img = cv::imread("image.jpg");
//Konvertálás HSV tartományba
Mat hsv;
cv::cvtColor(img, hsv, cv::COLOR_BGR2HSV);
//Szűrés kék színre
Mat filtered;
cv::inRange(hsv, Scalar(100,10,25), Scalar(130,255,255), filtered);
//Eredmény kiralyzolása
cv::imshow("Color filter",filtered);
cv::waitKey(0);
```
>kép az eredményről

[Learn more](https://docs.opencv.org/3.4/da/d97/tutorial_threshold_inRange.html)

### Morfológiai transzformációk
A morfológiai transzformációk az alakzatok formájára vannak hatással. Valamilyen struktúráló elemet kombinálnak halmazműveletekkel, az eredmény kiszámítása során. Használhatóak zajcsökkentésre, alakzatok elkülönítésére, intenzitás csúcsok és lyukak megtalálására. Bináris képeken (fekete-fehér) szoktuk őket alkalmazni.
#### Dilation
A művelet során egy tetszőleges méretű és alakú kernellel (általában négyzet, vagy kör) végig megyünk a képen (itt ez a struktúráló elem). Az anchor pointot minden pixelre helyezve megkeressük a kernel által fedett területen a maximális pixelintenzitást, és azzal helyettesítjük az anchor point-ban lévő pixelt. Ez azt eredményezi, hogy a képen lévő világos területek kitágulnak.
Az OpenCV természetesn biztosít hozzá függvényt:
```c++
void cv::dilate(InputArray src, OutputArray dst, InputArray kernel, Point anchor = Point(-1,-1), int iterations = 1, int borderType = BORDER_CONSTANT, const Scalar& borderValue = morphologyDefaultBorderValue()); 
```
- **src**: bemeneti kép, akármennyi csatornája lehet (ezeket egymástól függetlenül dolgozza fel), mélysége (depth) ezek valamelyike lehet: `CV_8U`, `CV_16U`, `CV_16S`, `CV_32F` or `CV_64F`
- **dst**: kimeneti kép, ugyanolyan méretű és típusú kell legyen, mint a bemeneti
- **kernel**: a struktúráló elem, amit a dilation-höz használ, a [getStructuringElement](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#gac342a1bb6eabf6f55c803b09268e36dc) segítségével kérhető le. Ha `Mat()`, akkor egy 3x3 négyzetet használ.
- **anchor**: a kernel melyik pontjában legyen az anchor, alapértelmezetten a közepe
- **iterations**: hányszor iteráljon végig a képen
- **borderType**: pixel extrapoláció módja, [részletek itt](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga209f2f4869e304c82d07739337eae7c5) (a kép széléről lelógó kernelt hogyan kezelje)
- **borderValue**: konstans határ esetén annak az értéke

[További info](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#ga4ff0f3318642c4f469d0e11f242f3b6c)
```c++
//Kép beolvasása
Mat img = cv::imread("image.jpg", IMREAD_GRAYSCALE);
//Hely előkészítése és struktúráló lekérése
Mat dst;
Mat element = getStructuringElement(MORPH_RECT, Size(5,5));
dilate(src, dst, element);
//Eredmény kiralyzolása
cv::imshow("Erosion", dst);
cv::waitKey(0);
```
>KÉP AZ EREDMÉNYRŐL

#### Erosion
A dilation-höz hasonló, lényegében annak az ellentéte. Ugyan úgy működik, viszont maximum helyett a kernel által fedett terület minimumát választja az anchor point új értékének. Ezzel elvékonyítja a világos területek a képen.
```c++
void cv::erode  (InputArray src, OutputArray dst, InputArray kernel, Point anchor = Point(-1,-1), int iterations = 1, int borderType = BORDER_CONSTANT, const Scalar& borderValue = morphologyDefaultBorderValue()); 
```
- **src**: bemeneti kép, akármennyi csatornája lehet (ezeket egymástól függetlenül dolgozza fel), mélysége (depth) ezek valamelyike lehet: `CV_8U`, `CV_16U`, `CV_16S`, `CV_32F` or `CV_64F`
- **dst**: kimeneti kép, ugyanolyan méretű és típusú kell legyen, mint a bemeneti
- **kernel**: a struktúráló elem, amit az erosion-höz használ, a [getStructuringElement](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#gac342a1bb6eabf6f55c803b09268e36dc) segítségével kérhető le. Ha `Mat()`, akkor egy 3x3 négyzetet használ.
- **anchor**: a kernel melyik pontjában legyen az anchor, alapértelmezetten a közepe
- **iterations**: hányszor iteráljon végig a képen
- **borderType**: pixel extrapoláció módja, [részletek itt](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga209f2f4869e304c82d07739337eae7c5) (a kép széléről lelógó kernelt hogyan kezelje)
- **borderValue**: konstans határ esetén annak az értéke
[További info](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#gaeb1e0c1033e3f6b891a25d0511362aeb)

```c++
//Kép beolvasása
Mat img = cv::imread("image.jpg", IMREAD_GRAYSCALE);
//Hely előkészítése és struktúráló lekérése
Mat dst;
Mat element = getStructuringElement(MORPH_ELLIPSE, Size(5,5));
erode(src, dst, element);
//Eredmény kiralyzolása
cv::imshow("Erosion", dst);
cv::waitKey(0);
```
>KÉP AZ EREDMÉNYRŐL

[Learn more](https://docs.opencv.org/3.4/db/df6/tutorial_erosion_dilatation.html)

Ennek a két morfológiai művelet kombinálásával további összetett műveletek végezhetők el, melyekhez ugyancsak beépített támogatást nyújt az OpenCV. Ezeket már 1 függvénnyel végezhetjük el, paraméterrel jelölve a konkrét műveletet.
```c++
void cv::morphologyEx (InputArray src, OutputArray dst, int op, InputArray kernel, Point anchor = Point(-1,-1), int iterations = 1, int borderType = BORDER_CONSTANT, const Scalar& borderValue = MorphologyDefaultBorderValue());
```
- **src**: bemeneti kép, akármennyi csatornája lehet (ezeket egymástól függetlenül dolgozza fel), mélysége (depth) ezek valamelyike lehet: `CV_8U`, `CV_16U`, `CV_16S`, `CV_32F` or `CV_64F`
- **dst**: kimeneti kép, ugyanolyan méretű és típusú kell legyen, mint a bemeneti
- **op**: a végrehajtandó összetett művelet, [ezek közül](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#ga7be549266bad7b2e6a04db49827f9f32) lehet választani
- **kernel**: a struktúráló elem, amit az erosion-höz használ, a [getStructuringElement](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#gac342a1bb6eabf6f55c803b09268e36dc) segítségével kérhető le. Ha `Mat()`, akkor egy 3x3 négyzetet használ.
- **anchor**: a kernel melyik pontjában legyen az anchor, alapértelmezetten a közepe
- **iterations**: hányszor iteráljon végig a képen
- **borderType**: pixel extrapoláció módja, [részletek itt](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga209f2f4869e304c82d07739337eae7c5) (a kép széléről lelógó kernelt hogyan kezelje)
- **borderValue**: konstans határ esetén annak az értéke

```c++
PÉLDA KÓD
```

[További info](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#ga67493776e3ad1a3df63883829375201f)

#### Opening
Egy képre erosion és dilation egymás utáni alkalmazása. Hasznos, hogy a kis objektumokat eltüntesse a képről (általában fekete hátteren fehér objektumok létrehozására törekszünk).

  $dst = dilation(erosion(src, kernel))$

![Opening](/pictures/morphology/opening.jpg "Az opening hatása")
#### Closing
Az opening fordítottja, először a dilation-t hajtja végre, utána az erosion-t. Kicsi lyukak eltávolítására használjuk (fekete pöttyök).

$dst = erosion(dilation(src, kernel))$

![Erosion](/pictures/morphology/closing.jpg "A closing hatása")
#### Morphological Gradient
A dilation és erosion különbsége. Alakzatok körvonalát (outline) lehet vele meghatározni.

$dst = dilation(src, kernel) - erosion(src, kernel)$

![Gradient](/pictures/morphology/gradient.jpg "A morphological gradient hatása")
[Learn more](https://docs.opencv.org/3.4/d3/dbe/tutorial_opening_closing_hats.html)

### Kép simítás
A simítást (smoothing) vagy elmosást (blur) leginkább zaj eltávolítására használjuk (ha úgy tetszik a nagyfrekvenciás zajt szűri ki a képről). Ehhez (szokás szerint) egy kernellel végigmászunk a kép összes pixelén, ami a környezetben lévő pixelek súlyát tartalmazza. Az ilyen súlyozott átlagot képző szűrpket nevezzük *lineáris szűrőknek*. Az alapján, hogy a kernelbe a súlyokat hogyan határozzuk meg, többféle szűrőt különböztetünk meg.
#### Normalizált doboz szűrő
(normalized box filter)
A legegyszerűbb szűrő, a számtani közepét veszi a környező pixeleknek (a kernelben az összes pixel súlya.

$K = {1\over K_{width} \cdot K_{height}}
    \begin{bmatrix}
    1 & 1 & ... & 1\\
    1 & 1 & ... & 1\\
    . & . & ... & .\\
    1 & 1 & ... & 1
    \end{bmatrix}
$

Az OpenCV rendelkezik ehhez beépített függvénnyel:
```c++
void cv::blur(InputArray src, OutputArray dst, Size ksize, Point anchor = Point(-1,-1), int borderType = BORDER_DEFAULT);
```
- **src**: bemeneti kép, akármennyi csatornája lehet (ezeket egymástól függetlenül dolgozza fel), mélysége (depth) ezek valamelyike lehet: `CV_8U`, `CV_16U`, `CV_16S`, `CV_32F` or `CV_64F`
- **dst**: kimeneti kép, ugyanolyan méretű és típusú kell legyen, mint a bemeneti
- **ksize**: a kernel mérete
- **anchor**: a kernel pontja, amit minden pixelre illeszt (alapértelmezetten a közepe)
- **borderType**: pixel extrapoláció módja, [részletek itt](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga209f2f4869e304c82d07739337eae7c5) (a kép széléről lelógó kernelt hogyan kezelje)
[További info](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#ga8c45db9afe636703801b0b2e440fce37)

```c++
PÉLDA KÓD
```

>KÉP AZ EREDMÉNYRŐL

#### Medián szűrő
A pixelt a kernel által meghatározott környezetében lévő pixelértékek mediánjára cseréli le.
Az OpenCV függvénye:
```c++
void cv::medianBlur(InputArray src, OutputArray dst, int ksize);
```
- **src**: bemeneti kép, akármennyi csatornája lehet (ezeket egymástól függetlenül dolgozza fel), mélysége (depth) ezek valamelyike lehet: `CV_8U`, `CV_16U`, `CV_16S`, `CV_32F` or `CV_64F`
- **dst**: kimeneti kép, ugyanolyan méretű és típusú kell legyen, mint a bemeneti
- **ksize**: a kernel mérete
- a kép szélénél a `BORDER_REPLICATE` extrapolációt használja ([BorderTypes](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga209f2f4869e304c82d07739337eae7c5))
[További info](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#ga564869aa33e58769b4469101aac458f9)

```c++
PÉLDA KÓD
```

>KÉP AZ EREDMÉNYRŐL

#### Gaussian Blur
Talán a leghasznosabb szűrő, viszont lassabb. A kernel értékeit Gauss-eloszlás szerint töltik ki (a közepén a legnagyobb a súlyozás, a széle felé pedig egyre jobban lecsökken). Ezt mutatja az alábbi két ábra, egy 1D-s és egy 2D-s Gauss-eloszlás ábrázolása (függőleges tengelyen a súlyok):

![1D Gauss distribution](/pictures/morphology/1d_gauss.png "1D-s Gauss-eloszlás")![2D Gauss distribution](/pictures/morphology/2d_gauss.png "2D-s Gauss-eloszlás")

OpenCV függvény:
```c++
void cv::GaussianBlur(InputArray src, OutputArray dst, Size ksize, double, sigmaX, double sigmaY = 0, int borderType = BORDER_DEFAULT);
```
- **src**: bemeneti kép, akármennyi csatornája lehet (ezeket egymástól függetlenül dolgozza fel), mélysége (depth) ezek valamelyike lehet: `CV_8U`, `CV_16U`, `CV_16S`, `CV_32F` or `CV_64F`
- **dst**: kimeneti kép, ugyanolyan méretű és típusú kell legyen, mint a bemeneti
- **ksize**: a kernel mérete, a szélesség és magasság eltérhet, de pozitív páratlan számoknak kell lennie. Vagy ha nullák, akkor a sigma-ból számolódik
- **sigmaX**: a Gauss-eloszlás szórása X irányba
- **sigmaY**: a Gauss-eloszlás szórása Y irányba. Ha 0, akkor sigmaX-ből számolódik. Ha mindkettő 0, akkor a ksize alapján ([részletek](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#gac05a120c1ae92a6060dd0db190a61afa)).
- **borderType**: pixel extrapoláció módja, [részletek itt](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga209f2f4869e304c82d07739337eae7c5) (a kép széléről lelógó kernelt hogyan kezelje)
[További info](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#gaabe8c836e97159a9193fb0b11ac52cf1)

```c++
PÉLDA KÓD
```

>KÉP AZ EREDMÉNYRŐL

#### Bilateral Filter
Az eddigi szűrők a kép simítás során nem csak a zajt távolítják el a képről, de az eléket is ezzel együtt elmossák. Ezt egy bizonyos szintig csökkenthetjük ezzel a szűrővel.
A súlyozás jelen esetben 2 részletből ál. Az első komponens ugyan az, mint a Gauss-szűrő esetében. A második figyelembe veszi az intenzitás beli különbséget a vizsgált és a szomszédos pixel között. ([Bővebben](http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/MANDUCHI1/Bilateral_Filtering.html))

OpenCV függvény:
```c++
void cv::GaussianBlur(InputArray src, OutputArray dst, int d, double sigmaColor, double sigmaSpace = 0, int borderType = BORDER_DEFAULT);
```
- **src**: bemeneti kép, akármennyi csatornája lehet (ezeket egymástól függetlenül dolgozza fel), mélysége (depth) ezek valamelyike lehet: `CV_8U`, `CV_16U`, `CV_16S`, `CV_32F` or `CV_64F`
- **dst**: kimeneti kép, ugyanolyan méretű és típusú kell legyen, mint a bemeneti
- **d**: a kernel átmérője, ha nem pozitív, akkor a sigmaSpace-ből számolódik
>A nagy szűrők (d>5) nagyon lassúak, ezért kb real-time alkalmazáshoz 5, offline alkalmazáshoz 9 az ajánlott érték (ahol erős szűrés kell)

- **sigmaColor**: nagyobb értéke azt eredményezi, hogy a (színtérben) távolabbi színek a pixel környezetéből össze mosódnak, ezáltal nagyobb kb azonos színű területeket eredményezve
- **sigmaSpace**: nagyobb értéke azt jelenti, hogy a messzebbi pixelek befolyásolják egymást, amég a színük elég közel van egymáshoz (->sigmaColor)
>Az egyszerűség kedvéért a 2 szigma érték egyenlő lehet. Ha alacsonyak (<10), akkor a szűrőnek ekvésbé lesz erőteljes hatása, ha nagyok (>150), akkor viszont igen, a kép "rajzfilmszerűen" fog kinézni

- **borderType**: pixel extrapoláció módja, [részletek itt](https://docs.opencv.org/3.4/d2/de8/group__core__array.html#ga209f2f4869e304c82d07739337eae7c5) (a kép széléről lelógó kernelt hogyan kezelje)
[További info](https://docs.opencv.org/3.4/d4/d86/group__imgproc__filter.html#ga9d7064d478c95d60003cf839430737ed)

```c++
PÉLDA KÓD
```

>KÉP AZ EREDMÉNYRŐL

[Learn more](https://docs.opencv.org/3.4/dc/dd3/tutorial_gausian_median_blur_bilateral_filter.html)

### Él- és körvonal keresés
[imgproc modul](https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html)


#### Canny Edge Detection
######(John F. Canny, 1986)
Optimális felismerőként is ismert. Három kritériumot próbál teljesíteni:
- csak tényleg létező éleket találjon meg (alacsony hiba arány)
- az észlelt él és valós él pixeleinek a távolsága minimális legyen (jó lokalizáció)
- egy élre csak egyszer jelezzen

##### Lépések:
1. zaj szűrés Gauss-szűrővel
2. A kép intenzitás gradiensének kiszámolása ([Sobel operátor](https://en.wikipedia.org/wiki/Sobel_operator))
3. Nem-maximum elnyomás: eltávolítja azokat a pixeleket amik nem az él részei, vékonyabb vonalakat eredményezve
4. Hiszterézis:
  a) Ha a pixel gradiens magasabb a felső határnál: elfogadjuk élként
  b) Ha a pixel gradiens alacsonyabb az alsó határnál: visszautasítjuk
  c) Ha a kettő között van: csak akkor fogadjuk el ha valamelyik szomszédját elfogadtuk. Canny 2:1 és 3:1 közötti arányt javasolt határoknak

```c++
void cv::Canny(InputArray image, OutputArray edges, double threshold1, double  threshold2, int apertureSize = 3, bool L2gradient = false);
```
- **image:** 8 bites kép
- **edges:** egy csatornás 8 bites kép, ami tartalmazza az éleket
- **threshold1:** az egyik threshold a hiszterézishez
- **threshold2:** a másik threshold a hiszterézishez
- **apertureSize:** a gradiens számító kernel mérete
- **L2gradient:** flag annak a jelzésére, hogy kell-e a pontosabb gradiens számolás

[További info](https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html#ga04723e007ed888ddf11d9ba04e2232de)

```c++
PÉLDA KÓD
```

[Learn more](https://docs.opencv.org/3.4/da/d5c/tutorial_canny_detector.html)

#### Hough Line
Ez a transzformáció egyenes vonalak detektálására használható. Ahhoz, hogy alkalmazni tudjuk előtte egy élkeresést érdemes elvégezni.
##### Működése:
Általánosan egy egyenest a  `y = a * x + b` egyenlettel adhatunk meg. Ezt polár koordináta rendszerben felírva az alább egyenlet jön ki:

![Polar-coordinate line](/figures/polar_line.jpg "Egy egyenes ábrázolása polár koordináta rendszerben, normálvektor segítségével")

$y = -\frac{cos(\theta)}{sin(\theta)} \cdot x +  \frac{r}{sin(\theta)}$

Ahol theta az egyenes normálvektora és az x tengely által bezárt szög, r pedig az egyenes távolsága az origótól.

Ezt rendezhetjük a $r = x \cdot cos(\theta) + y \cdot sin(\theta)$ alakra

  1. Általánosságban az $(x_0;y_0)$ ponton átmenő egyenesek a
  $r = x_0 \cdot cos(\theta) + y_0 \cdot sin(\theta)$
  egyenlettel írhatók le, ahol minden egyes $(r;\theta)$ pár egy konkrét egyenes.
  2. Tehát az 1 ponton átmenő egyenesek theta és r értékét egy szinuszfüggvényként ábrázolhatjuk:
  ![Line sinusoid](/figures/line_sinusoid.jpg "Egy ponton átmenő egyenesek theta-r párosainak ábrázolása")
  Ennek a függvényne tehát minden pontja 1 egyenest jelent.
  3. Hogyha ezt a kép több pontjára is elvégezzük, és a kapott függvényeket ábrázoljuk, akkor az alábbi képet kapjuk:
  ![Multiple polar-coordinate line](/figures/multiple_line_sinusoid.jpg "Több különböző képponton átmenő egyenesek")
  A különböző függvények tehát a különböző pontokon átmenő theta-r párosok. Ahol ezek metszik egymást, az azzal a theta-r párral leírható egyenes mind a két ponton átmegy. Tehát minél több ilyen metszés van, annál valószínűbb, hogy az a theta-r pár tényleg egy létező egyenes.

A HoughLine tehát nyilvántartja, hogy egy theta-r értéknél hány pont esetében volt metszés, és ha az egy beállított `threshold`-ot meghalad, akkor egy képen szereplő egyenesnek tekinti.
*Persze ha ezt az összes pontra így elvégeznénk mindenhol egyenest találna. Ezért van szükség előtte az élkeresésre, mivel annak a bináris kép kimenete lesz a HoughLines bemenete, így az csak a fehér képpontokra végzi el a kiértékelést, ahol az élek vannak.*

##### OpenCV függvény:

```c++
void cv::HoughLines( InputArray image, OutputArray lines, double rho, double theta, int threshold, double srn = 0, double stn = 0, double min_theta = 0, double max_theta = V_PI);
```
- **image:** 8-bites, egy csatornás bináris kép. Módosulhat a futás során
- **lines:** A megtalált vonalak. Minden vonál 2 vagy 3 elemű vektor: `(r,theta)`, vagy `(r, theta, votes)`
    + r: az origótól (bal felső sarok, (0,0)) való távolság
    + theta: egyenes x-tengellyel bezárt szöge (0~függőleges - pi/2~vízszintes)
    + votes: hány képpontban megy ott át egyenes
- **rho:** r felbontása pixelben
- **theta:** theta felbontása radiánban
- **threshold:** a minimális szavazatok száma, ahhoz, hogy egyenesnek tekintse
- **srn:** több skálájú Hough transzformáció esetén az `rho` osztója
- **stn:** több skálájú Hough transzformáció esetén az `theta` osztója
    + ha `srn`˙és `stn` is 0 akkor a klasszikus Hough transzformációt végzi, egyébként pedig pozitívnak kell lennie
- **min_theta:** a keresett egyenesek minimális x tengellyel bezárt szöge. 0 és `max_theta` közé kell essen
- **max_theta:** a keresett egyenesek maximális x tengellyel bezárt szöge. `max_theta` és `CV_PI` közé kell essen

[Valószínőség alapú Hough line transzformáció](https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html#ga8618180a5948286384e3b7ca02f6feeb)
```c++
PÉLDA KÓD
```
>KÉP AZ EREDMÉNYRŐL

[Learn more](https://docs.opencv.org/3.4/d9/db0/tutorial_hough_lines.html)

#### Hough Circles



[Learn more](https://docs.opencv.org/3.4/d4/d70/tutorial_hough_circle.html)
#### Find Contours
[További info](https://docs.opencv.org/3.4/d3/dc0/group__imgproc__shape.html#ga17ed9f5d79ae97bd4c7cf18403e1689a)
[Learn more](https://docs.opencv.org/3.4/df/d0d/tutorial_find_contours.html)
## Rajzolás, GUI
### Alacsony szintű rajzolás
[imgproc modul](https://docs.opencv.org/3.4/dd/d1a/group__imgproc__feature.html)
#### Vonal
[További info](https://docs.opencv.org/3.4/d6/d6e/group__imgproc__draw.html#ga7078a9fae8c7e7d13d24dac2520ae4a2)
#### Elipszis
[További info](https://docs.opencv.org/3.4/d6/d6e/group__imgproc__draw.html#ga28b2267d35786f5f890ca167236cbc69)
#### Téglalap
[További info](https://docs.opencv.org/3.4/d6/d6e/group__imgproc__draw.html#ga07d2f74cadcf8e305e810ce8eed13bc9)
#### Kör
[További info](https://docs.opencv.org/3.4/d6/d6e/group__imgproc__draw.html#gaf10604b069374903dbd0f0488cb43670)
#### Kitöltött sokszog
[További info](https://docs.opencv.org/3.4/d6/d6e/group__imgproc__draw.html#gaf30888828337aa4c6b56782b5dfbd4b7)

[Learn more](https://docs.opencv.org/3.4/d3/d96/tutorial_basic_geometric_drawing.html)
#### Draw contours
[További info](https://docs.opencv.org/3.4/d6/d6e/group__imgproc__draw.html#ga746c0625f1781f1ffc9056259103edbc)
[Learn more](https://docs.opencv.org/3.4/df/d0d/tutorial_find_contours.html)
### GUI elemek
[highgui modul](https://docs.opencv.org/3.4/d4/dd5/highgui_8hpp.html)
#### Trackbar
[További info](https://docs.opencv.org/3.4/d7/dfc/group__highgui.html#gaf78d2155d30b728fc413803745b67a9b)

##### Minimum beállítása
[További info](https://docs.opencv.org/3.4/d7/dfc/group__highgui.html#ga67d73c4c9430f13481fd58410d01bd8d)
##### Maximum beállítása
[További info](https://docs.opencv.org/3.4/d7/dfc/group__highgui.html#ga7e5437ccba37f1154b65210902fc4480)
##### Pozíció lekérdezés
[További info](https://docs.opencv.org/3.4/d7/dfc/group__highgui.html#ga122632e9e91b9ec06943472c55d9cda8)
##### Pozíció beállítás
[További info](https://docs.opencv.org/3.4/d7/dfc/group__highgui.html#ga67d73c4c9430f13481fd58410d01bd8d)

[Learn more](https://docs.opencv.org/3.4/da/d6a/tutorial_trackbar.html)
#### Egér kezelés

[Learn more](http://opencvexamples.blogspot.com/2014/01/detect-mouse-clicks-and-moves-on-image.html)


[Még több tutorial](https://docs.opencv.org/3.4/d9/df8/tutorial_root.html)



https://markdownlivepreview.com/
https://github.com/adam-p/markdown-here/wiki/Markdown-Here-Cheatsheet

**TODO: kigyűjteni a linkeket**
[imgproc modul]:
[highgui modul]:
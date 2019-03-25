---
layout: post
title: C# - string
published: true
categories: [.NET]
tags: c# net string
---
## 문자열의 마지막에 있는 \r\n 제거 하기
  
```
Text2.TrimEnd(Environment.NewLine.ToCharArray());
```
  
  
  
## 지정 단어로 문자열 분해
  
```
string readLine = "A=B";
string[] word = readLine.Split(new Char[] { '=' });

string[] stringSeparators = new string[] {"[#@#]"};
var TokenString = logdata.Split(stringSeparators, StringSplitOptions.RemoveEmptyEntries);
```
  
  
  
##문자열 포맷
  
```
string CheatString = string.Format("@timeevent1:{0}", textBoxBlessing.Text);
```
  
  
  
## string.Format 에서 " 사용하기
  
```
string.Format("Hi \"{0}\"", "C#");
```
  
  
  
## 지정된 단어를 문자열에서 (제일 처음 나오는)제거 하고 싶을 때
  
```
string str1 = "C:\ASDD";
string str2 = "C:\ASDD\Server.exe";
// str2에서 str1을 제거하고 싶음
string str3 = str2.Trim( str1.ToCharArray() );
```
  
  
  
## 문자열 "C\Text\Sub" 에서 "Sub"만 추출 하고 싶을 때
  
```
textBox1.Text = "C\Text\Sub";
int index = textBox1.Text.LastIndexOf(@"\");
string CurVersionFoldName = textBox1.Text.Substring(index + 1);
```
  
  
  
## 바이트 길이 알기
  
```
int iAsciiLength = ((byte[])Encoding.GetEncoding(0).GetBytes(stringData)).Length;
```
  
  
  
## Performance Tips: Faster than StringBuilder?
- http://blogs.msdn.com/b/fyuan/archive/2012/08/14/performance-tips-faster-than-stringbuilder.aspx
  
```
const int sLen = 30, Loops = 5000;  
System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();  
string sSource = new String('X', sLen);
string sDest = "";  
// Time string concatenation.
//  
watch.Restart();
for (int i = 0; i < Loops; i++) sDest += sSource;
watch.Stop();
Console.WriteLine("Concatenation took {0,8:N3} ms. {1} chars", watch.Elapsed.TotalMilliseconds, sDest.Length);  
// Time StringBuilder.
//
watch.Restart();
System.Text.StringBuilder sb = new System.Text.StringBuilder(sLen * Loops);
for (int i = 0; i < Loops; i++) sb.Append(sSource);
sDest = sb.ToString();
watch.Stop();
Console.WriteLine("StringBuilder took {0,8:N3} ms. {1} chars", watch.Elapsed.TotalMilliseconds, sDest.Length);  
// Time String.Join.
//
watch.Restart();
string[] list = new string[Loops];
for (int i = 0; i < Loops; i++) list[i] = sSource;
sDest = String.Join(String.Empty, list, 0, Loops);
watch.Stop();
Console.WriteLine("String.Join took {0,8:N3} ms. {1} chars", watch.Elapsed.TotalMilliseconds, sDest.Length);
```
  
<pre>
결과
Concatenation took 524.630 ms. 150000 chars
StringBuilder took 0.426 ms.   150000 chars
String.Join   took 0.310 ms.   150000 chars
</pre>
  
  
  
## String.Join
- 문자열 배열을 구분자 문자로 연결하고 싶은 경우 사용한다.
- 구분자를 String.Empty를 사용하면 그냥 문자열 배열을 연결할 수도 있다.
  
```
string[] stArrayData = {"8", "ASD", "XYZ"};

// 배열 내의 데이터를 모두 , 로 구분하면서 연결한다
string stCsvData = string.Join(",", stArrayData);
```
  
  
  
## List를 string으로 변환
  
```
List<int> result = new List<int();

string.Join(",", result.ToArray());
```
  
  
  
## String과 StringBuilder의 차이점
- http://drgt.tistory.com/10
- StringBuilder의 경우 동일 메모리 주소에 문자열을 추가함으로써 재할당에 드는 불필요한 메모리 공간 낭비를 줄인다
- 문자열 상수연산이나 5개 이하의 문자열 연산에서는 StringBuilder보다는 +연산자를 사용하는 것이 바람직
- StringBuilder를 써야 할 경우는 다수의 문자열 연산을 하거나 반복문 내에서 문자열 연산을 수행할 경우
- 사용 예
    - http://lifehack.kr/90020114219
    - http://blog.naver.com/vieng/130172857489
  
  
  
## 영어, 한글, 숫자 조사
  
```
// 영문자이면 true
if ((0x61 <= ch && ch <= 0x7A) || (0x41 <= ch && ch <= 0x5A))
{
    return true;
}

// 숫자이면 true
if (0x30 <= ch && ch <= 0x39)
{
    return true;
}

// 한글이면 true
if (char.GetUnicodeCategory(ch) == System.Globalization.UnicodeCategory.OtherLetter)
{
    return true;
}
```
  
  
  
## 완성형 한글인지 조사
  
``
public static bool IsCompleteKorean(string name)
{
    // 완성형 검사
    string useTextKr = "가각간갇갈갉갊감갑값갓갔강갖갗같갚갛개객갠갤갬갭갯갰갱갸갹갼걀걋걍걔걘걜거걱건걷걸걺검겁것겄겅겆겉겊겋게겐겔겜겝겟겠겡겨격겪견겯결겸겹겻겼경곁계곈곌곕곗고곡곤곧골곪곬곯곰곱곳공곶과곽관괄괆괌괍괏광괘괜괠괩괬괭괴괵괸괼굄굅굇굉교굔굘굡굣구국군굳굴굵굶굻굼굽굿궁궂궈궉권궐궜궝궤궷귀귁귄귈귐귑귓규균귤그극근귿글긁금급긋긍긔기긱긴긷길긺김깁깃깅깆깊까깍깎깐깔깖깜깝깟깠깡깥깨깩깬깰깸깹깻깼깽꺄꺅꺌꺼꺽꺾껀껄껌껍껏껐껑께껙껜껨껫껭껴껸껼꼇꼈꼍꼐꼬꼭꼰꼲꼴꼼꼽꼿꽁꽂꽃꽈꽉꽐꽜꽝꽤꽥꽹꾀꾄꾈꾐꾑꾕꾜꾸꾹꾼꿀꿇꿈꿉꿋꿍꿎꿔꿜꿨꿩꿰꿱꿴꿸뀀뀁뀄뀌뀐뀔뀜뀝뀨끄끅끈끊끌끎끓끔끕끗끙끝끼끽낀낄낌낍낏낑나낙낚난낟날낡낢남납낫났낭낮낯낱낳내낵낸낼냄냅냇냈냉냐냑냔냘냠냥너넉넋넌널넒넓넘넙넛넜넝넣네넥넨넬넴넵넷넸넹녀녁년녈념녑녔녕녘녜녠노녹논놀놂놈놉놋농높놓놔놘놜놨뇌뇐뇔뇜뇝뇟뇨뇩뇬뇰뇹뇻뇽누눅눈눋눌눔눕눗눙눠눴눼뉘뉜뉠뉨뉩뉴뉵뉼늄늅늉느늑는늘늙늚늠늡늣능늦늪늬늰늴니닉닌닐닒님닙닛닝닢다닥닦단닫달닭닮닯닳담답닷닸당닺닻닿대댁댄댈댐댑댓댔댕댜더덕덖던덛덜덞덟덤덥덧덩덫덮데덱덴델뎀뎁뎃뎄뎅뎌뎐뎔뎠뎡뎨뎬도독돈돋돌돎돐돔돕돗동돛돝돠돤돨돼됐되된될됨됩됫됴두둑둔둘둠둡둣둥둬뒀뒈뒝뒤뒨뒬뒵뒷뒹듀듄듈듐듕드득든듣들듦듬듭듯등듸디딕딘딛딜딤딥딧딨딩딪따딱딴딸땀땁땃땄땅땋때땍땐땔땜땝땟땠땡떠떡떤떨떪떫떰떱떳떴떵떻떼떽뗀뗄뗌뗍뗏뗐뗑뗘뗬또똑똔똘똥똬똴뙈뙤뙨뚜뚝뚠뚤뚫뚬뚱뛔뛰뛴뛸뜀뜁뜅뜨뜩뜬뜯뜰뜸뜹뜻띄띈띌띔띕띠띤띨띰띱띳띵라락란랄람랍랏랐랑랒랖랗래랙랜랠램랩랫랬랭랴략랸럇량러럭런럴럼럽럿렀렁렇레렉렌렐렘렙렛렝려력련렬렴렵렷렸령례롄롑롓로록론롤롬롭롯롱롸롼뢍뢨뢰뢴뢸룀룁룃룅료룐룔룝룟룡루룩룬룰룸룹룻룽뤄뤘뤠뤼뤽륀륄륌륏륑류륙륜률륨륩륫륭르륵른를름릅릇릉릊릍릎리릭린릴림립릿링마막만많맏말맑맒맘맙맛망맞맡맣매맥맨맬맴맵맷맸맹맺먀먁먈먕머먹먼멀멂멈멉멋멍멎멓메멕멘멜멤멥멧멨멩며멱면멸몃몄명몇몌모목몫몬몰몲몸몹못몽뫄뫈뫘뫙뫼묀묄묍묏묑묘묜묠묩묫무묵묶문묻물묽묾뭄뭅뭇뭉뭍뭏뭐뭔뭘뭡뭣뭬뮈뮌뮐뮤뮨뮬뮴뮷므믄믈믐믓미믹민믿밀밂밈밉밋밌밍및밑바박밖밗반받발밝밞밟밤밥밧방밭배백밴밸뱀뱁뱃뱄뱅뱉뱌뱍뱐뱝버벅번벋벌벎범법벗벙벚베벡벤벧벨벰벱벳벴벵벼벽변별볍볏볐병볕볘볜보복볶본볼봄봅봇봉봐봔봤봬뵀뵈뵉뵌뵐뵘뵙뵤뵨부북분붇불붉붊붐붑붓붕붙붚붜붤붰붸뷔뷕뷘뷜뷩뷰뷴뷸븀븃븅브븍븐블븜븝븟비빅빈빌빎빔빕빗빙빚빛빠빡빤빨빪빰빱빳빴빵빻빼빽뺀뺄뺌뺍뺏뺐뺑뺘뺙뺨뻐뻑뻔뻗뻘뻠뻣뻤뻥뻬뼁뼈뼉뼘뼙뼛뼜뼝뽀뽁뽄뽈뽐뽑뽕뾔뾰뿅뿌뿍뿐뿔뿜뿟뿡쀼쁑쁘쁜쁠쁨쁩삐삑삔삘삠삡삣삥사삭삯산삳살삵삶삼삽삿샀상샅새색샌샐샘샙샛샜생샤샥샨샬샴샵샷샹섀섄섈섐섕서석섞섟선섣설섦섧섬섭섯섰성섶세섹센셀셈셉셋셌셍셔셕션셜셤셥셧셨셩셰셴셸솅소속솎손솔솖솜솝솟송솥솨솩솬솰솽쇄쇈쇌쇔쇗쇘쇠쇤쇨쇰쇱쇳쇼쇽숀숄숌숍숏숑수숙순숟술숨숩숫숭숯숱숲숴쉈쉐쉑쉔쉘쉠쉥쉬쉭쉰쉴쉼쉽쉿슁슈슉슐슘슛슝스슥슨슬슭슴습슷승시식신싣실싫심십싯싱싶싸싹싻싼쌀쌈쌉쌌쌍쌓쌔쌕쌘쌜쌤쌥쌨쌩썅써썩썬썰썲썸썹썼썽쎄쎈쎌쏀쏘쏙쏜쏟쏠쏢쏨쏩쏭쏴쏵쏸쐈쐐쐤쐬쐰쐴쐼쐽쑈쑤쑥쑨쑬쑴쑵쑹쒀쒔쒜쒸쒼쓩쓰쓱쓴쓸쓺쓿씀씁씌씐씔씜씨씩씬씰씸씹씻씽아악안앉않알앍앎앓암압앗았앙앝앞애액앤앨앰앱앳앴앵야약얀얄얇얌얍얏양얕얗얘얜얠얩어억언얹얻얼얽얾엄업없엇었엉엊엌엎에엑엔엘엠엡엣엥여역엮연열엶엷염엽엾엿였영옅옆옇예옌옐옘옙옛옜오옥온올옭옮옰옳옴옵옷옹옻와왁완왈왐왑왓왔왕왜왝왠왬왯왱외왹왼욀욈욉욋욍요욕욘욜욤욥욧용우욱운울욹욺움웁웃웅워웍원월웜웝웠웡웨웩웬웰웸웹웽위윅윈윌윔윕윗윙유육윤율윰윱윳융윷으윽은을읊음읍읏응읒읓읔읕읖읗의읜읠읨읫이익인일읽읾잃임입잇있잉잊잎자작잔잖잗잘잚잠잡잣잤장잦재잭잰잴잼잽잿쟀쟁쟈쟉쟌쟎쟐쟘쟝쟤쟨쟬저적전절젊점접젓정젖제젝젠젤젬젭젯젱져젼졀졈졉졌졍졔조족존졸졺좀좁좃종좆좇좋좌좍좔좝좟좡좨좼좽죄죈죌죔죕죗죙죠죡죤죵주죽준줄줅줆줌줍줏중줘줬줴쥐쥑쥔쥘쥠쥡쥣쥬쥰쥴쥼즈즉즌즐즘즙즛증지직진짇질짊짐집짓징짖짙짚짜짝짠짢짤짧짬짭짯짰짱째짹짼쨀쨈쨉쨋쨌쨍쨔쨘쨩쩌쩍쩐쩔쩜쩝쩟쩠쩡쩨쩽쪄쪘쪼쪽쫀쫄쫌쫍쫏쫑쫓쫘쫙쫠쫬쫴쬈쬐쬔쬘쬠쬡쭁쭈쭉쭌쭐쭘쭙쭝쭤쭸쭹쮜쮸쯔쯤쯧쯩찌찍찐찔찜찝찡찢찧차착찬찮찰참찹찻찼창찾채책챈챌챔챕챗챘챙챠챤챦챨챰챵처척천철첨첩첫첬청체첵첸첼쳄쳅쳇쳉쳐쳔쳤쳬쳰촁초촉촌촐촘촙촛총촤촨촬촹최쵠쵤쵬쵭쵯쵱쵸춈추축춘출춤춥춧충춰췄췌췐취췬췰췸췹췻췽츄츈츌츔츙츠측츤츨츰츱츳층치칙친칟칠칡침칩칫칭카칵칸칼캄캅캇캉캐캑캔캘캠캡캣캤캥캬캭컁커컥컨컫컬컴컵컷컸컹케켁켄켈켐켑켓켕켜켠켤켬켭켯켰켱켸코콕콘콜콤콥콧콩콰콱콴콸쾀쾅쾌쾡쾨쾰쿄쿠쿡쿤쿨쿰쿱쿳쿵쿼퀀퀄퀑퀘퀭퀴퀵퀸퀼큄큅큇큉큐큔큘큠크큭큰클큼큽킁키킥킨킬킴킵킷킹타탁탄탈탉탐탑탓탔탕태택탠탤탬탭탯탰탱탸턍터턱턴털턺텀텁텃텄텅테텍텐텔템텝텟텡텨텬텼톄톈토톡톤톨톰톱톳통톺톼퇀퇘퇴퇸툇툉툐투툭툰툴툼툽툿퉁퉈퉜퉤튀튁튄튈튐튑튕튜튠튤튬튱트특튼튿틀틂틈틉틋틔틘틜틤틥티틱틴틸팀팁팃팅파팍팎판팔팖팜팝팟팠팡팥패팩팬팰팸팹팻팼팽퍄퍅퍼퍽펀펄펌펍펏펐펑페펙펜펠펨펩펫펭펴편펼폄폅폈평폐폘폡폣포폭폰폴폼폽폿퐁퐈퐝푀푄표푠푤푭푯푸푹푼푿풀풂품풉풋풍풔풩퓌퓐퓔퓜퓟퓨퓬퓰퓸퓻퓽프픈플픔픕픗피픽핀필핌핍핏핑하학한할핥함합핫항해핵핸핼햄햅햇했행햐향허헉헌헐헒험헙헛헝헤헥헨헬헴헵헷헹혀혁현혈혐협혓혔형혜혠혤혭호혹혼홀홅홈홉홋홍홑화확환활홧황홰홱홴횃횅회획횐횔횝횟횡효횬횰횹횻후훅훈훌훑훔훗훙훠훤훨훰훵훼훽휀휄휑휘휙휜휠휨휩휫휭휴휵휸휼흄흇흉흐흑흔흖흗흘흙흠흡흣흥흩희흰흴흼흽힁히힉힌힐힘힙힛힝";

    for (int i = 0; i < name.Length; ++i)
    {
        var c = name[i];
        if (char.GetUnicodeCategory(c) != System.Globalization.UnicodeCategory.OtherLetter)
        {
            continue;
        }

        if (useTextKr.Contains(c) == false)
        {
            return false;
        }
    }

    return true;
}
```
  
  
  
## UTF-8로 변환
  
```
str= "testé";
byte[] utf8Bytes = Encoding.UTF8.GetBytes(str);
return Encoding.UTF8.GetString(utf8Bytes);
```
  
  
  
## 라이브러리
### StringFormatEx
- 기존의 String.Format과 호환
- Fast, small, lightweight. Even with all these features, Smart.Format performs nearly as well as String.Format -- creating output in microseconds
- Pluralization
    - { Number : plural(lang) : singular | plural | more... }
      
```
Smart.Format("There {0:plural:is 1 item|are {} items}.", number);
// outputs "There is 1 item." or "There are 2 items."

Smart.Format("You have {0} new {0:message|messages}", emails.Count)
```
  
```
Smart.Format("{Name} commented on {Gender:his|her|their} photo", user)
```
  
- Lists
    - { IEnumerable : alias : template | spacer | finalSpacer }
  
```
Smart.Format("{Users:{Name}|, |, and } liked your comment", users)
// Outputs: "Jim, Pam, Dwight, and Michael liked your comment"

var items = new[] {"one", "two", "three"};
Smart.Format("{0:list:{}|, |, and}", items);
// "one, two, and three"
```
  
- Choose (conditional logic)
    - { Any Value : choose(1|2|3) : output 1 | output 2 | output 3 | default }
  
```
Smart.Format("{0:choose(1|2|3):one|two|three|other}", someNumber)
Smart.Format("{0:choose(True|False):yes|no}", someBoolean)
Smart.Format("{0:choose(Male|Female):his|her|their}", genderEnum)
Smart.Format("{0:choose(null): N/A | {} }", valueOrNull)

Smart.Format("You have {Messages.Count:choose(0|1):no new messages|a new message|{} new messages}", messages)
```
  
- Named placeholders
  
```
Smart.Format("{Name} from {Address.City}, {Address.State}", user)
Smart.Format("{Prop.ToString}", new{ Prop = 999 })
```
  
```
Smart.Format("{0} {1}", "Hello", "World");
// Outputs: "Hello World"

Smart.Format("{0:N3} | {1:MMMM, yyyy}", 5.5, new DateTime(2010,3,4));
// Outputs: "5.500 | March, 2010"

Smart.Format("|{0,-10}|{1,10}|", "Left", "Right");
// Outputs: "|Left      |     Right|"

Smart.Format("{{0}} {{{0}}} {{}}", "Zero");
// Outputs: "{0} {Zero} {}"

Smart.Format(CultureInfo.GetCultureInfo("en-US"), "{0:C}", 1234) // Outputs: "$1,234.00"
Smart.Format(CultureInfo.GetCultureInfo("ja-JP"), "{0:C}", 1234) // Outputs: "¥1234"
```
  
  
### ObjectToString
- https://github.com/shanselman/ObjectToString
  
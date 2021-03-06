<properties
	pageTitle="Azure AD Connect Sync: 関数参照 | Microsoft Azure"
	description="Azure AD Connect Sync での宣言型のプロビジョニングの式の参照"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="StevenPo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/07/2016"
	ms.author="andkjell;markusvi"/>


# Azure AD Connect Sync: 関数リファレンス


Azure Active Directory Sync では、同期時の属性値を操作するために関数を使用します。関数の構文は、次の形式を使用して表されます。`<output type> FunctionName(<input type> <position name>, ..)`

関数がオーバーロード状態の場合に複数の構文を受け入れると、すべての有効な構文が一覧表示されます。関数は厳密に型指定され、渡された型が文書化された型と一致することを確認します。型が一致しない場合は、エラーがスローされます。

型は次の構文で表されます。

- **bin** – バイナリ
- **bool** – ブール値
- **dt** – UTC の日付/時刻
- **enum** – 既知の定数の列挙
- **exp** – 評価結果にブール値が想定される式
- **mvbin** – 複数値のバイナリ
- **mvstr** – 複数値の参照
- **num** – 数値
- **ref** – 単一値の参照
- **str** – 単一値の文字列
- **var** – その他の (ほとんど) すべての型のバリアント
- **void** – 値を返しません



## 関数参照

----------
**換算:**

[CBool](#cbool) &nbsp;&nbsp;&nbsp;&nbsp; [CDate](#cdate) &nbsp;&nbsp;&nbsp;&nbsp; [CGuid](#cguid) &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [ConvertFromBase64](#convertfrombase64) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertToBase64](#converttobase64) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertFromUTF8Hex](#convertfromutf8hex) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertToUTF8Hex](#converttoutf8hex) &nbsp;&nbsp;&nbsp;&nbsp; [CNum](#cnum) &nbsp;&nbsp;&nbsp;&nbsp; [CRef](#cref) &nbsp;&nbsp;&nbsp;&nbsp; [CStr](#cstr) &nbsp;&nbsp;&nbsp;&nbsp; [StringFromGuid](#StringFromGuid) &nbsp;&nbsp;&nbsp;&nbsp; [StringFromSid](#stringfromsid)

**日付/時刻:**

[DateAdd](#dateadd) &nbsp;&nbsp;&nbsp;&nbsp; [DateFromNum](#datefromnum) &nbsp;&nbsp;&nbsp;&nbsp; [FormatDateTime](#formatdatetime) &nbsp;&nbsp;&nbsp;&nbsp; [Now](#now) &nbsp;&nbsp;&nbsp;&nbsp; [NumFromDate](#numfromdate)

**ディレクトリ**

[DNComponent](#dncomponent) &nbsp;&nbsp;&nbsp;&nbsp; [DNComponentRev](#dncomponentrev) &nbsp;&nbsp;&nbsp;&nbsp; [EscapeDNComponent](#escapedncomponent)

**評価:**

[IsBitSet](#isbitset) &nbsp;&nbsp;&nbsp;&nbsp; [IsDate](#isdate) &nbsp;&nbsp;&nbsp;&nbsp; [IsEmpty](#isempty) &nbsp;&nbsp;&nbsp;&nbsp; [IsGuid](#isguid) &nbsp;&nbsp;&nbsp;&nbsp; [IsNull](#isnull) &nbsp;&nbsp;&nbsp;&nbsp; [IsNullOrEmpty](#isnullorempty) &nbsp;&nbsp;&nbsp;&nbsp; [IsNumeric](#isnumeric) &nbsp;&nbsp;&nbsp;&nbsp; [IsPresent](#ispresent) &nbsp;&nbsp;&nbsp;&nbsp; [IsString](#isstring)

**算術:**

[BitAnd](#bitand) &nbsp;&nbsp;&nbsp;&nbsp; [BitOr](#bitor) &nbsp;&nbsp;&nbsp;&nbsp; [RandomNum](#randomnum)

**複数値**

[Contains](#contains) &nbsp;&nbsp;&nbsp;&nbsp; [Count](#count) &nbsp;&nbsp;&nbsp;&nbsp; [Item](#item) &nbsp;&nbsp;&nbsp;&nbsp; [ItemOrNull](#itemornull) &nbsp;&nbsp;&nbsp;&nbsp; [Join](#join) &nbsp;&nbsp;&nbsp;&nbsp; [RemoveDuplicates](#removeduplicates) &nbsp;&nbsp;&nbsp;&nbsp; [Split](#split)

**プログラム フロー:**

[Error](#error) &nbsp;&nbsp;&nbsp;&nbsp; [IIF](#iif) &nbsp;&nbsp;&nbsp;&nbsp; [Switch](#switch)


**テキスト**

[GUID](#guid) &nbsp;&nbsp;&nbsp;&nbsp; [InStr](#instr) &nbsp;&nbsp;&nbsp;&nbsp; [InStrRev](#instrrev) &nbsp;&nbsp;&nbsp;&nbsp; [LCase](#lcase) &nbsp;&nbsp;&nbsp;&nbsp; [Left](#left) &nbsp;&nbsp;&nbsp;&nbsp; [Len](#len) &nbsp;&nbsp;&nbsp;&nbsp; [LTrim](#ltrim) &nbsp;&nbsp;&nbsp;&nbsp; [Mid](#mid) &nbsp;&nbsp;&nbsp;&nbsp; [PadLeft](#padleft) &nbsp;&nbsp;&nbsp;&nbsp; [PadRight](#padright) &nbsp;&nbsp;&nbsp;&nbsp; [PCase](#pcase) &nbsp;&nbsp;&nbsp;&nbsp; [Replace](#replace) &nbsp;&nbsp;&nbsp;&nbsp; [ReplaceChars](#replacechars) &nbsp;&nbsp;&nbsp;&nbsp; [Right](#right) &nbsp;&nbsp;&nbsp;&nbsp; [RTrim](rtrim) &nbsp;&nbsp;&nbsp;&nbsp; [Trim](#trim) &nbsp;&nbsp;&nbsp;&nbsp; [UCase](#ucase) &nbsp;&nbsp;&nbsp;&nbsp; [Word](#word)

----------
### BitAnd

**説明:** BitAnd 関数は、値に指定のビットを設定します。

**構文:** `num BitAnd(num value1, num value2)`

- value1, value2: ともに AND になる数値

**解説:** この関数は両方のパラメーターをバイナリ表現に変換し、ビットを次に設定します。

- 0 - *マスク*と*フラグ*で対応するビットの 1 つまたは両方が 0 の場合。
- 1 - 対応するビットの両方が 1 の場合。

つまり、両方のパラメーターの対応するビットが 1 の場合を除くすべてのケースで 0 を返します。

**例:** `BitAnd(&HF, &HF7)` 16 進数の "F" AND "F7" は 7 と評価されるので、7 を返します。

----------
### BitOr

**説明:** BitOr 関数は、値に指定のビットを設定します。

**構文:** `num BitOr(num value1, num value2)`

- value1, value2: ともに OR になる数値

**解説:** この関数は両方のパラメーターをバイナリ表現に変換し、マスクとフラグで対応するビットの 1 つまたは両方が 1 の場合はビットを 1 に設定し、対応する両方のビットが 0 の場合は 0 に設定します。つまり、両方のパラメーターの対応するビットが 0 の場合を除くすべてのケースで 1 を返します。

----------
### CBool

**説明:** CBool 関数は、式の評価結果に基づいてブール値を返します。

**構文:** `bool CBool(exp Expression)`

**解説:** 式の評価結果が 0 以外の値の場合、CBool は True を返し、それ以外の場合は False を返します。

**例:** `CBool([attrib1] = [attrib2])`

両方の属性が同じ値を持つ場合は、True を返します。

----------
### CDate

**説明:** CDate 関数は、文字列から UTC DateTime を返します。DateTime は Sync ではネイティブの属性の型ではありませんが、一部の関数で使用されます。

**構文:** `dt CDate(str value)`

- Value: 日付、時刻、オプションでタイム ゾーンを含む文字列

**解説:** 文字列は常に UTC で返されます。

**例:** `CDate([employeeStartTime])` 従業員の作業開始時刻に基づいて、DateTime を返します。

`CDate("2013-01-10 4:00 PM -8")` "2013-01-11 12:00 AM" を表す DateTime を返します。

----------
### CGuid

**説明:** CGuid 関数は、GUID の文字列表現をそのバイナリ表現に変換します。

**構文:** `bin CGuid(str GUID)`

- このパターンで書式設定される文字列: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx または {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

----------
### 指定値を含む

**説明:** Contains 関数は、複数値の属性内で文字列を検索します。

**構文:** `num Contains (mvstring attribute, str search)` - 大文字小文字を区別 `num Contains (mvstring attribute, str search, enum Casetype)` `num Contains (mvref attribute, str search)` - 大文字小文字を区別

- attribute: 検索対象の複数値の属性。<br>
- search: 属性で検索する文字列。<br>
- Casetype: CaseInsensitive または CaseSensitive。<br>

文字列が見つかった複数値の属性にインデックスを返します。文字列が見つからない場合は 0 を返します。

**解説:** 複数値の文字列属性の場合、検索では、値で部分文字列が検出されます。参照属性の場合、一致と見なされるためには、検索された文字列は正確に値と一致する必要があります。

**例:** `IIF(Contains([proxyAddresses],"SMTP:")>0,[proxyAddresses],Error("No primary SMTP address found."))` proxyAddresses 属性にプライマリ メール アドレスが含まれている場合は (大文字の " SMTP:" で表されます) proxyAddress 属性を返し、それ以外の場合はエラーを返します。

----------
### ConvertFromBase64

**説明:** ConvertFromBase64 関数は、指定した base64 でエンコードされた値を正規の文字列に変換します。

**構文:** `str ConvertFromBase64(str source)` - エンコードに Unicode を前提とします。<br> `str ConvertFromBase64(str source, enum Encoding)`

- source: Base64 でエンコードされた文字列  
- Encoding: Unicode、ASCII、UTF8

**例** `ConvertFromBase64("SABlAGwAbABvACAAdwBvAHIAbABkACEA")` `ConvertFromBase64("SGVsbG8gd29ybGQh", UTF8)`

どちらの例でも "*Hello world!*" を返します。

----------
### ConvertFromUTF8Hex

**説明:** ConvertFromUTF8Hex 関数は、指定した UTF8 の 16 進数でエンコードされた値を文字列に変換します。

**構文:** `str ConvertFromUTF8Hex(str source)`

- source: UTF8 の 2 バイトでエンコードされた文字列

**解説:** この関数と ConvertFromBase64(,UTF8) との違いは、結果が DN 属性で表示される点です。この形式は Azure Active Directory で DN として使用されます。

**例:** `ConvertFromUTF8Hex("48656C6C6F20776F726C6421")` "*Hello world!*" を返します。

----------
### ConvertToBase64

**説明:** ConvertToBase64 関数は、文字列を Unicode の base64 文字列に変換します。整数の配列の値を、base 64 桁でエンコードされているそれと同等の文字列表現に変換します。

**構文:** `str ConvertToBase64(str source)`

**例:** `ConvertToBase64("Hello world!")` "SABlAGwAbABvACAAdwBvAHIAbABkACEA" を返します。

----------
### ConvertToUTF8Hex

**説明:** ConvertToUTF8Hex 関数は、文字列を UTF8 の 16 進数でエンコードされた値に変換します。

**構文:** `str ConvertToUTF8Hex(str source)`

**解説:** この関数の出力形式は、DN 属性の形式として Azure Active Directory で使用されます。

**例:** `ConvertToUTF8Hex("Hello world!")` 48656C6C6F20776F726C6421 を返します。

----------
### カウント

**説明:** Count 関数は、複数値の属性内の要素数を返します。

**構文:** `num Count(mvstr attribute)`

----------
### CNum

**説明:** CNum 関数は、文字列を受け取り、数値データ型を返します。

**構文:** `num CNum(str value)`

----------
### CRef

**説明:** 文字列を参照属性に変換します。

**構文:** `ref CRef(str value)`

**例:** `CRef("CN=LC Services,CN=Microsoft,CN=lcspool01,CN=Pools,CN=RTC Service," & %Forest.LDAP%)`

----------
### CStr

**説明:** CStr 関数は、文字列データ型に変換します。

**構文:** `str CStr(num value)` `str CStr(ref value)` `str CStr(bool value)`

- value: 数値、参照属性、ブール値を指定できます。

**例:** `CStr([dn])` "cn=Joe,dc=contoso,dc=com" を返すことができます。

----------
### DateAdd

**説明:** 指定した時間間隔が追加された日付を含む Date を返します。

**構文:** `dt DateAdd(str interval, num value, dt date)`

- interval: 追加する時間間隔を表す文字列式。文字列には次のいずれかの値が必要です。
 - yyyy: 年
 - q: 四半期
 - m: 月
 - y: 年間通算日
 - d: 日
 - w: 週日
 - ww: 週
 - h: 時
 - n: 分
 - s: 秒
- value: 追加する単位の数。正の数 (将来の日時を取得する場合) または負の数 (過去の日時を取得する場合) を指定できます。
- date: 間隔が追加される日付を表す DateTime。

**例:** `DateAdd("m", 3, CDate("2001-01-01"))` 3 か月を追加した結果の "2001-04-01" を表す DateTime を返します。

----------
### DateFromNum

**説明:** DateFromNum 関数は、AD の日付形式の値を DateTime 型に変換します。

**構文:** `dt DateFromNum(num value)`

**例:** `DateFromNum([lastLogonTimestamp])` `DateFromNum(129699324000000000)` 2012-01-01 23:00:00 を表す DateTime を返します。

----------
### DNComponent

**説明:** DNComponent 関数は、指定した DN コンポーネントの値を左から返します。

**構文:** `str DNComponent(ref dn, num ComponentNumber)`

- dn: 解釈する参照属性
- ComponentNumber: 返される DN のコンポーネント

**例:** `DNComponent([dn],1)` dn が "cn=Joe,ou=…," の場合は、Joe が返されます。

----------
### DNComponentRev

**説明:** DNComponentRev 関数は、指定した DN コンポーネントの値を右 (端) から返します。

**構文:** `str DNComponentRev(ref dn, num ComponentNumber)` `str DNComponentRev(ref dn, num ComponentNumber, enum Options)`

- dn: 解釈する参照属性
- ComponentNumber - 返される DN のコンポーネント
- Options: DC – "dc =" ですべてのコンポーネントを無視します。

**例:** dn が "cn=Joe,ou=Atlanta,ou=GA,ou=US, dc=contoso,dc=com" の場合、`DNComponentRev([dn],3)` `DNComponentRev([dn],1,"DC")` 両方が US を返します。

----------
### エラー

**説明:** Error 関数を使用して、カスタム エラーを返します。

**構文:** `void Error(str ErrorMessage)`

**例:** `IIF(IsPresent([accountName]),[accountName],Error("AccountName is required"))` 属性 accountName が存在しない場合は、オブジェクトでエラーをスローします。

----------
### EscapeDNComponent

**説明:** EscapeDNComponent 関数は、DN のコンポーネントを 1 つ受け取り、LDAP で表すためにそれをエスケープします。

**構文:** `str EscapeDNComponent(str value)`

**例:** `EscapeDNComponent("cn=" & [displayName]) & "," & %ForestLDAP%)` displayName 属性に LDAP でエスケープする必要のある文字が含まれている場合でも、LDAP ディレクトリでオブジェクトを作成できることを確認します。

----------
### FormatDateTime

**説明:** FormatDateTime 関数を使用して、DateTime を指定した形式の文字列に設定します。

**構文:** `str FormatDateTime(dt value, str format)`

- value: DateTime 形式の値
- format: 変換する形式を表す文字列。

**解説:** 形式に使用できる値については、「[ユーザー定義の日付/時刻書式 (Format 関数)](http://msdn2.microsoft.com/library/73ctwf33(VS.90).aspx)」を参照してください。

**例:**

`FormatDateTime(CDate("12/25/2007"),"yyyy-mm-dd")` 結果は "2007-12-25" です。

`FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")` "20140905081453.0Z" などの結果になります。

----------
### GUID

**説明:** GUID 関数は、新しいランダムな GUID を生成します。

**構文:** `str GUID()`

----------
### IIF

**説明:** IIF 関数は、指定した条件に基づいて使用できる値セットのうち、いずれかを返します。

**構文:** `var IIF(exp condition, var valueIfTrue, var valueIfFalse)`

- condition: 評価結果が true または false になる任意の値または式。
- valueIfTrue: 条件の評価結果が true になる場合に返される値。
- valueIfFalse: 条件の評価結果が false になる場合に返される値。

**例:** `IIF([employeeType]="Intern","t-" & [alias],[alias])` ユーザーがインターンの場合はユーザーのエイリアスの先頭に "t-" を付けて返し、そうでない場合はユーザーのエイリアスをそのまま返します。

----------
### InStr

**説明:** InStr 関数は文字列内の最初の部分文字列を検索します。

**構文:**

`num InStr(str stringcheck, str stringmatch)` `num InStr(str stringcheck, str stringmatch, num start)` `num InStr(str stringcheck, str stringmatch, num start , enum compare)`

- stringcheck: 検索対象の文字列
- stringmatch: 検出対象の文字列
- start: 部分文字列の検索開始位置
- compare: vbTextCompare または vbBinaryCompare

**解説:** 部分文字列が見つかった位置を返します。見つからなかった場合は 0 を返します。

**例:** `InStr("The quick brown fox","quick")` 評価結果は 5 になります。

`InStr("repEated","e",3,vbBinaryCompare)` 評価結果は 7 になります。

----------
### InStrRev

**説明:** InStrRev 関数は文字列内の最後の部分文字列を検索します。

**構文:** `num InstrRev(str stringcheck, str stringmatch)` `num InstrRev(str stringcheck, str stringmatch, num start)` `num InstrRev(str stringcheck, str stringmatch, num start, enum compare)`

- stringcheck: 検索対象の文字列
- stringmatch: 検出対象の文字列
- start: 部分文字列の検索開始位置
- compare: vbTextCompare または vbBinaryCompare

**解説:** 部分文字列が見つかった位置を返します。見つからなかった場合は 0 を返します。

**例:** `InStrRev("abbcdbbbef","bb")` 7 を返します。

----------
### IsBitSet

**説明:** IsBitSet 関数は、ビットが設定されているかどうかをテストします。

**構文:** `bool IsBitSet(num value, num flag)`

- value: 評価対象の数値。flag: 評価対象のビットがある数値。

**例:** `IsBitSet(&HF,4)` ビット "4" が 16 進数値 "F" で設定されているため True が返されます。

----------
### IsDate

**説明:** 式が DateTime 型として評価できる場合、IsDate 関数の評価結果は True になります。

**構文:** `bool IsDate(var Expression)`

**解説:** CDate() が成功するかどうかを判断するために使用します。

----------
### IsEmpty

**説明:** 属性が CS または MV に存在しても評価結果が空の文字列である場合、IsEmpty 関数の評価結果は True になります。

**構文:** `bool IsEmpty(var Expression)`

----------
### IsGuid

**説明:** 文字列が GUID に変換できる場合、IsGUID 関数の評価結果は true になります。

**構文:** `bool IsGuid(str GUID)`

**解説:** GUID は次のいずれかのパターンに従った文字列として定義されます: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx または {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

CGuid() が成功するかどうかを判断するために使用します。

**例:** `IIF(IsGuid([strAttribute]),CGuid([strAttribute]),NULL)` StrAttribute に GUID 形式がある場合はバイナリ表現を返します。それ以外の場合は Null を返します。

----------
### IsNull

**説明:** 式の評価結果が Null の場合、IsNull 関数は true を返します。

**構文:** `bool IsNull(var Expression)`

**解説:** 属性の場合、Null は属性の不在によって表されます。

**例:** `IsNull([displayName])` CS または MV に属性がない場合は True を返します。

----------
### IsNullOrEmpty

**説明:** IsNullOrEmpty 関数は、式が null または空の文字列の場合、true を返します。

**構文:** `bool IsNullOrEmpty(var Expression)`

**解説:** 属性の場合は、属性がないか、存在しても空の文字列の場合、評価結果は True になります。<br> この関数の逆の関数は IsPresent です。

**例:** `IsNullOrEmpty([displayName])` CS または MV に属性がないか、空の文字列の場合は True を返します。

----------
### IsNumeric

**説明:** IsNumeric 関数は、式を数値型として評価できるかどうかを示すブール値を返します。

**構文:** `bool IsNumeric(var Expression)`

**解説:** CNum() が式の解析に成功するかどうかを判断するために使用します。

----------
### IsString

**説明:** 式を string 型として評価できる場合、IsString 関数の評価結果は True になります。

**構文:** `bool IsString(var expression)`

**解説:** CStr() が式の解析に成功するかどうかを判断するために使用します。

----------
### IsPresent

**説明:** 式の評価結果が Null でもなく、空でもない文字列の場合、IsPresent 関数は true を返します。

**構文:** `bool IsPresent(var expression)`

**解説:** この関数の逆の関数は IsNullOrEmpty です。

**例:** `Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`

----------
### 項目

**説明:** Item 関数は複数値の文字列/属性から 1 つの項目を返します。

**構文:** `var Item(mvstr attribute, num index)`

- attribute: 複数値の属性
- index: 複数値の文字列内の項目へのインデックス。

**解説:** Contains 関数は複数値の属性内の項目に対するインデックスを返すため、Item 関数を Contains 関数と共に使用すると便利です。

インデックスが範囲外にある場合は、エラーをスローします。

**例:** `Mid(Item([proxyAddress],Contains([proxyAddress], "SMTP:")),6)` プライマリ メール アドレスを返します。

----------
### ItemOrNull

**説明:** ItemOrNull 関数は複数値の文字列/属性から 1 つの項目を返します。

**構文:** `var ItemOrNull(mvstr attribute, num index)`

- attribute: 複数値の属性
- index: 複数値の文字列内の項目へのインデックス。

**解説:** Contains 関数は複数値の属性内の項目に対するインデックスを返すため、ItemOrNull 関数を Contains 関数と共に使用すると便利です。

インデックスが範囲外にある場合は、Null 値を返します。

----------
### Join

**説明:** Join 関数は、複数値の文字列を受け取り、指定した区切り記号が項目間に挿入された、単一値の文字列を返します。

**構文:** `str Join(mvstr attribute)` `str Join(mvstr attribute, str Delimiter)`

- attribute: 結合対象の文字列が含まれる複数値の属性。
- delimiter: 返される文字列内で部分文字列を区切るために使用する任意の文字列。省略した場合は、空白文字 (" ") が使用されます。delimiter が長さ 0 の文字列 ("") または Nothing の場合、リスト内のすべての項目は、区切り記号なしで連結されます。

**解説:** Join 関数と Split 関数の間には類似点があります。Join 関数は、文字列の配列を受け取り、区切り文字列を使用してそれらを結合し、単一の文字列を返します。Split 関数は、文字列を受け取って区切り記号で分割し、文字列の配列を返します。ただし、Join 関数が任意の区切り文字列を使った文字列を連結できるのに対し、Split 関数で文字列を分割する際には 1 文字の区切り記号しか使用できないという大きな違いがあります。

**例:** `Join([proxyAddresses],",")` "SMTP:john.doe@contoso.com,smtp:jd@contoso.com" を返すことができます

----------
### LCase

**説明:** LCase 関数は、文字列内のすべての文字を小文字に変換します。

**構文:** `str LCase(str value)`

**例:** `LCase("TeSt")` "test" を返します。

----------
### Left

**説明:** Left 関数は文字列の左端から数えて指定した文字数分の文字を返します。

**構文:** `str Left(str string, num NumChars)`

- string: 返される文字を含む文字列
- NumChars: 文字列の左端から数えて返される文字数を指定する数値

**解説:** 文字列内の最初の numChars 文字分の文字を含む文字列。

- numChars = 0 の場合、空の文字列を返します。
- numChars < 0 の場合、入力文字列を返します。
- string が null の場合、空の文字列を返します。

文字列に含まれる文字数が numChars で指定した数より少ない場合は、文字列と同一の文字列 (パラメーター 1 のすべての文字が含まれる) が返されます。

**例:** `Left("John Doe", 3)` "Joh" を返します。

----------
### Len

**説明:** Len 関数は文字列の文字数を返します。

**構文:** `num Len(str value)`

**例:** `Len("John Doe")` 8 を返します。

----------
### LTrim

**説明:** LTrim 関数は文字列の先頭の空白文字を削除します。

**構文:** `str LTrim(str value)`

**例:** `LTrim(" Test ")` "Test " を返します。

----------
### Mid

**説明:** Mid 関数は文字列の指定した位置から数えて、指定した文字数を返します。

**構文:** `str Mid(str string, num start, num NumChars)`

- string: 返される文字を含む文字列
- start: 文字列内で返される文字の開始位置を指定する数値
- NumChars: 文字列の位置から数えて返される文字数を指定する数値

**解説:** 文字列内の start 位置から数えて numChars 文字分の文字を返します。文字列内の start 位置から数えて numChars 文字分の文字が含まれる文字列。

- numChars = 0 の場合、空の文字列を返します。
- numChars < 0 の場合、入力文字列を返します。
- start > 文字列の長さの場合、入力文字列を返します。
- start < = 0 の場合、入力文字列を返します。
- string が null の場合、空の文字列を返します。

文字列で start 位置から後に numChar 文字が残っていない場合、返すことのできるすべての文字が返されます。

**例:** `Mid("John Doe", 3, 5)` "hn Do" を返します。

`Mid("John Doe", 6, 999)` "Doe" を返します

----------
### Now

**説明:** Now 関数は、コンピューターのシステムの日付と時刻に従って、現在の日付と時刻を指定する DateTime を返します。

**構文:** `dt Now()`

----------
### NumFromDate

**説明:** NumFromDate 関数は、AD の日付形式の日付を返します。

**構文:** `num NumFromDate(dt value)`

**例:** `NumFromDate(CDate("2012-01-01 23:00:00"))` 129699324000000000 を返します。

----------
### PadLeft

**説明:** PadLeft 関数は、指定した埋め込み文字を使用して、指定した長さになるまで左側に文字列を埋め込みます。

**構文:** `str PadLeft(str string, num length, str padCharacter)`

- string: 埋め込む文字列。
- length: 文字列の必要な長さを表す整数。
- padCharacter: 埋め込み文字として使用する 1 文字で構成された文字列。

**解説:**

- 文字列の長さが length 未満の場合、長さが length と等しくなるまで文字列の左端に padCharacter が繰り返し追加されます。
- PadCharacter には空白文字を指定できますが、null 値を指定することはできません。
- 文字列の長さが length 以上の場合、文字列は変更されない状態で返されます。
- 文字列に length 以上の長さがある場合、文字列と同一の文字列が返されます。
- 文字列の長さが length 未満の場合、padCharacter が埋め込まれた文字列が含まれる、必要な長さの新しい文字列が返されます。
- string が null の場合、この関数は空の文字列を返します。

**例:** `PadLeft("User", 10, "0")` “000000User” を返します。

----------
### PadRight

**説明:** PadRight 関数は、指定した埋め込み文字を使用して、指定した長さになるまで右側に文字列を埋め込みます。

**構文:** `str PadRight(str string, num length, str padCharacter)`

- string: 埋め込む文字列。
- length: 文字列の必要な長さを表す整数。
- padCharacter: 埋め込み文字として使用する 1 文字で構成された文字列。

**解説:**

- 文字列の長さが length 未満の場合、長さが length と等しくなるまで文字列の右端に padCharacter が繰り返し追加されます。
- padCharacter には空白文字を指定できますが、null 値を指定することはできません。
- 文字列の長さが length 以上の場合、文字列は変更されない状態で返されます。
- 文字列に length 以上の長さがある場合、文字列と同一の文字列が返されます。
- 文字列の長さが length 未満の場合、padCharacter が埋め込まれた文字列が含まれる、必要な長さの新しい文字列が返されます。
- string が null の場合、この関数は空の文字列を返します。

**例:** `PadRight("User", 10, "0")` “User000000” を返します。

----------
### PCase

**説明:** PCase 関数は、文字列内のスペースで区切られた単語の最初の文字を大文字に変換し、その他のすべての文字を小文字に変換します。

**構文:** `String PCase(string)`

**例:** `PCase("TEsT")` "Test" を返します。

----------
### RandomNum

**説明:** RandomNum 関数は、指定した間隔を空けた乱数を返します。

**構文:** `num RandomNum(num start, num end)`

- start: 生成するランダムな値の下限を指定する数値
- end: 生成するランダムな値の上限を指定する数値

**例:** `Random(100,999)` 734 を返すことができます。

----------
### RemoveDuplicates

**説明:** RemoveDuplicates 関数は複数値の文字列を受け取り、各値が一意であることを確認します。

**構文:** `mvstr RemoveDuplicates(mvstr attribute)`

**例:** `RemoveDuplicates([proxyAddresses])` 重複する値がすべて削除された後の、校正済み proxyAddress 属性を返します。

----------
### Replace

**説明:** Replace 関数は、見つかった文字列をすべて別の文字列に置き換えます。

**構文:** `str Replace(str string, str OldValue, str NewValue)`

- string: 値を置換する文字列。
- OldValue: 検索し、置換される文字列。
- NewValue: 置換する文字列。

**解説:** この関数は次の特殊なモニカーを認識します。

- \\n – 新しい行
- \\r – キャリッジ リターン
- \\t – タブ

**例:** `Replace([address],"\r\n",", ")` CRLF をコンマとスペースで置き換え、"One Microsoft Way, Redmond, WA, USA" とすることができます。

----------
### ReplaceChars

**説明:** ReplaceChars 関数は、ReplacePattern 文字列に見つかったすべての文字を置き換えます。

**構文:** `str ReplaceChars(str string, str ReplacePattern)`

- string: 文字を置換する文字列。
- ReplacePattern: 置換する文字のディクショナリが含まれる文字列。

形式は {source1}:{target1},{source2}:{target2},{sourceN},{targetN} です。source は検索対象の文字、target は置換する文字列です。

**解説:**

- この関数は定義した source の見つかった位置を受け取り、target と置き換えます。
- source は、厳密に 1 文字 (unicode) である必要があります。
- source は、空にすることも 2 文字以上にすることもできません (解析エラー)。
- target には、ö:oe、β:ss などの複数の文字を指定できます。
- target を空にして、文字を削除する必要があることを指定できます。
- source は、大文字と小文字を区別し、完全に一致する必要があります。
- 「,」(コンマ) と「:」(コロン) は予約文字であり、この関数を使用して置き換えることはできません。
- ReplacePattern 文字列内のスペースやその他の空白文字は無視されます。

**例:** `%ReplaceString% = ’:,Å:A,Ä:A,Ö:O,å:a,ä:a,ö,o`

`ReplaceChars("Räksmörgås",%ReplaceString%)` Raksmorgas を返します。

`ReplaceChars("O’Neil",%ReplaceString%)` "ONeil" を返します。1 つのチェックマークを削除するように定義されています。

----------
### Right

**説明:** Right 関数は文字列の右端から数えて指定した文字数分の文字を返します。

**構文:** `str Right(str string, num NumChars)`

- string: 返される文字を含む文字列
- NumChars: 文字列の右端から数えて返される文字数を指定する数値

**解説:** 文字列の last 位置から数えて NumChars 文字分の文字が返されます。

文字列内の最後の numChars 文字分の文字を含む文字列。

- numChars = 0 の場合、空の文字列を返します。
- numChars < 0 の場合、入力文字列を返します。
- string が null の場合、空の文字列を返します。

文字列に含まれる文字数が NumChars に指定した数より少ない場合は、文字列と同一の文字列が返されます。

**例:** `Right("John Doe", 3)` "Doe" を返します。

----------
### RTrim

**説明:** RTrim 関数は文字列の末尾の空白文字を削除します。

**構文:** `str RTrim(str value)`

**例:** `RTrim(" Test ")` " Test" を返します。

----------
### Split

**説明:** Split 関数は区切り記号で区切られた文字列を受け取り、複数値の文字列にします。

**構文:** `mvstr Split(str value, str delimiter)` `mvstr Split(str value, str delimiter, num limit)`

- value: 分離する区切り文字が含まれる文字列。
- delimiter: 区切り記号として使用される 1 文字。
- limit: 返される値の最大数。

**例:** `Split("SMTP:john.doe@contoso.com,smtp:jd@contoso.com",",")` proxyAddress 属性に有用な 2 つの要素が含まれる複数値の文字列を返します。

----------
### StringFromGuid

**説明:** StringFromGuid 関数は、バイナリ GUID を受け取り、文字列に変換します。

**構文:** `str StringFromGuid(bin GUID)`

----------
### StringFromSid

**説明:** StringFromSid 関数は、セキュリティ識別子が含まれるバイト配列または複数値のバイト配列を、文字列または複数値の文字列に変換します。

**構文:** `str StringFromSid(bin ObjectSID)` `mvstr StringFromSid(mvbin ObjectSID)`

----------
### Switch

**説明:** Switch 関数は、評価された条件に基づいて 1 つの値を返すために使用します。

**構文:** `var Switch(exp expr1, var value1[, exp expr2, var value … [, exp expr, var valueN]])`

- expr: 評価する必要のあるバリアント型の式。
- value: 対応する式が True の場合に返される値。

**解説:** Switch 関数の引数リストは、式と値のペアで構成されます。式は左から右に評価され、評価結果が True になる最初の式に関連付けられている値が返されます。各部分が正しくペアリングされていないと、実行時エラーが発生します。

たとえば、expr1 が True の場合、Switch は value1 を返します。たとえば、expr1 が False でも expr2 が True の場合、Switch は value2 を返します。

次の場合、Switch は Nothing を返します。

- すべての式が True でない場合。
- 最初の True 式に、Null である対応する値がある場合。

Switch は、返される式が 1 つであってもすべての式を評価します。このため、望ましくない影響が発生しないよう注意する必要があります。たとえば、式の評価結果が 0 除算のエラーの場合、エラーが発生します。

値には、カスタム文字列を返す Error 関数を指定することもできます。

**例:** `Switch([city] = "London", "English", [city] = "Rome", "Italian", [city] = "Paris", "French", True, Error("Unknown city"))` 主要都市で話される言語を返します。それ以外の場合はエラーを返します。

----------
### Trim

**説明:** Trim 関数は、文字列の先頭と末尾の空白文字を削除します。

**構文:** `str Trim(str value)` `mvstr Trim(mvstr value)`

**例:** `Trim(" Test ")` "Test" を返します。

`Trim([proxyAddresses])` proxyAddress 属性の値ごとに先頭と末尾の空白を削除します。

----------
### UCase

**説明:** UCase 関数は文字列内のすべての文字を大文字に変換します。

**構文:** `str UCase(str string)`

**例:** `UCase("TeSt")` "TEST" を返します。

----------
### Word

**説明:** Word 関数は、使用する区切り記号を記述するパラメーターと、返す単語の番号に基づいて、文字列内に含まれる単語を返します。

**構文:** `str Word(str string, num WordNumber, str delimiters)`

- string: 返される単語を含む文字列。
- WordNumber: 返すべき単語の番号を指定する数値。
- delimiters: 単語を識別するために使用される区切り記号を表す文字列

**解説:** 区切り記号のいずれかの文字で区切られた、文字列内の文字による各文字列が、単語として識別されます。

- num < 1 の場合、空の文字列を返します。
- string が null の場合、空の文字列を返します。

文字列に含まれる単語の数が指定より少ないか、区切り記号文字で識別されるどの単語も文字列に含まれていない場合は、空の文字列が返されます。

**例:** `Word("The quick brown fox",3," ")` "brown" を返します。

`Word("This,string!has&many separators",3,",!&#")` "has" を返します。

## その他のリソース

* [宣言型のプロビジョニングの式について](active-directory-aadconnectsync-understanding-declarative-provisioning-expressions.md)
* [Azure AD Connect Sync: 同期オプションのカスタマイズ](active-directory-aadconnectsync-whatis.md)
* [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0309_2016-->
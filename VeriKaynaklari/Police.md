<h1>Police</h1>
Poliçe kayıtlarına erişmek için kullanılır. Formüller kullanılarak poliçe üzerindeki tüm verilere (poliçe üzerindeki hali veya son durumu) erişilebilir.

<a href="../Tablolar/SPOLICE.md">SPOLICE</a> tablosunu kullanır.

<h2>İlişki Kurulabilecek Veri Kaynakları</h2>
<table>
<tr>
<th>Ana Veri Kaynağı</th>
<th>Ana Veri Kaynağı Kolon Adı</th>
<th>Çocuk Veri Kaynağı</th>
<th>Çocuk Veri Kaynağı Kolon Adi</th>
</tr>
<tr>
<td><a href="../VeriKaynaklari/Police.md">Police</a></td>
<td>PoliceKey</td>
<td><a href="../VeriKaynaklari/PoliceZeyil.md">PoliceZeyil</a></td>
<td>AnaPoliceKey</td>
</tr>
<tr>
<td><a href="../VeriKaynaklari/Police.md">Police</a></td>
<td>PoliceKey</td>
<td><a href="../VeriKaynaklari/PoliceAyrintiArac.md">PoliceAyrintiArac</a></td>
<td>PoliceKey</td>
</tr>
</table>


<h2>Power Query</h2>
<pre>
let
    config = let
        Source = Xml.Tables(File.Contents("C:\Power BI Raporlar\config.xml")),
        Table0 = Source{0}[Table],
        Table1 = Table.TransformColumnTypes(Table0,{{"server", type text}, {"database", type text}}),
        config = Table1{0},
        veritabani = Sql.Database(config[server], config[database]),
        #"Changed Type" = Table.TransformColumnTypes(Source,{{"server", type text}, {"database", type text}})
    in
        veritabani,
        dbo_SPOLICE = config{[Schema="dbo",Item="SPOLICE"]}[Data],
        #"Filtered Rows" = Table.SelectRows(dbo_SPOLICE, each ([IPT_KAYIT] = "I" or [IPT_KAYIT] = "K") and ([ZEYL_NO] = "   ")),
        #"Removed Other Columns" = Table.SelectColumns(#"Filtered Rows",{"YIL", "ACENTA", "BRANS", "POLICE_NO", "ZEYLKOD2", "ZEYL_KODU", "ZEYL_NO", "TECDIT_NO", "IPT_KAYIT", "BAS_TAR", "BIT_TAR", "TANZIM_TAR", "UW_YEAR", "TANZIM_YER", "KISI_SAYI", "PRIM", "TARIFE_BAS", "ACE_BOLGE_KODU", "IPTAL_DURUMU", "IPTAL_TARIHI", "ORTAK_NO", "ARAC_TARZ"}),
        #"Added Custom" = Table.AddColumn(#"Removed Other Columns", "PoliceKey", each [ACENTA]&"_"&[BRANS]&"_"&[POLICE_NO]&"_"&[TECDIT_NO]&"_"&[ZEYL_NO])
in
    #"Added Custom"
</pre>

<h2>Formüller</h2>

<h4>IptalEdilmis (Column)</h4>
Poliçenin iptal edilip edilmediğini belirtir.
<pre>IptalEdilmis = Police[IPTAL_DURUMU]=1</pre>

<h4>MebIptalEdilmis (Column)</h4>
Poliçenin mebdeinden iptal zeyili ile iptal edilip edilmediğini belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyil.md">PoliceZeyil</a>
<pre>MebIptalEdilmis = SUMMARIZE(TOPN(1; RELATEDTABLE(PoliceZeyil); PoliceZeyil[ZEYL_NO];DESC);[MebIptalZeyilimi])</pre>

<h4>YasayanPolice (Column)</h4>
Yaşayan poliçe olup olmadığını belirtir. 
<pre>YasayanPolice = AND(AND(Police[BAS_TAR]<=TODAY();TODAY()<=Police[BIT_TAR]);NOT(Police[IptalEdilmis]))</pre>

<h4>SonDurum_Tarz (Column)</h4>
Araç tarzının son durumunu belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyil.md">PoliceZeyil</a>
<pre>SonDurum_Tarz = SUMMARIZE(TOPN(1; RELATEDTABLE(PoliceZeyil); PoliceZeyil[ZEYL_NO];DESC);[ARAC_TARZ])</pre>

<h4>SonDurum_ZeyilNo (Column)</h4>
Poliçeye ait en son kesilmiş zeyilin numarasını belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyil.md">PoliceZeyil</a>
<pre>SonDurum_ZeyilNo = SUMMARIZE(TOPN(1; relatedtable(PoliceZeyil); PoliceZeyil[ZEYL_NO];DESC);[ZEYL_NO])</pre>

<h4>SonDurum_PlakaIlKodu (Column)</h4>
Poliçenin son durumundaki plaka il kodunu belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyil.md">PoliceZeyil</a>
<pre>SonDurum_PlakaIlKodu = SUMMARIZE(TOPN(1; RELATEDTABLE(PoliceZeyil); PoliceZeyil[ZEYL_NO];DESC);[PlakaIlKodu])</pre>

<h4>SonDurum_SigortaliIlKodu (Column)</h4>
Poliçenin son durumundaki sigortalı il kodunu belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilSigortali.md">PoliceZeyilSigortali</a>
<pre>SonDurum_SigortaliIlKodu = SUMMARIZE(TOPN(1; RELATEDTABLE(PoliceZeyilSigortali); PoliceZeyilSigortali[MZEYL_NO];DESC);[IL_KODU])</pre>

<h4>SonDurum_SigortaEttirenIlKodu (Column)</h4>
Poliçenin son durumundaki sigortalı il kodunu belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilSigortali.md">PoliceZeyilSigortali</a>
<pre>SonDurum_SigortaEttirenIlKodu = SUMMARIZE(TOPN(1; RELATEDTABLE(PoliceZeyilSigortaEttiren); PoliceZeyilSigortaEttiren[ZEYIL_NO];DESC);[IL_KODU])</pre>

<h4>SonDurum_RizikoIlKodu (Column)</h4>
Poliçenin son durumundaki sigortalı il kodunu belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilSigortali.md">PoliceZeyilSigortali</a>
<pre>SonDurum_RizikoIlKodu = SUMMARIZE(TOPN(1; RELATEDTABLE(PoliceZeyilRizikoAdresi); PoliceZeyilRizikoAdresi[MZEYL_NO];DESC);[IL_KODU])</pre>

<h4>X_TeminatVar (Column)</h4>
Bir poliçede (zeyillerde dikkate alınarak) X kodlu teminatın bulunup bulunmadığını belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilTeminat.md">PoliceZeyilTeminat</a>
<pre>X_TeminatVar = COUNTROWS(FILTER(RELATEDTABLE(PoliceZeyilTeminat);PoliceZeyilTeminat[TEM_KODU]="X")) > 0</pre>

<h4>PoliceKPK (Column)</h4>
Poliçenin zeyilleri de hesaba katılmış olarak net KPK rakamını belirtir. İptal olmuş poliçelerde rakam 0'dır.  İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyil.md">PoliceZeyil</a>
<pre>PoliceKPK = SUMX(PoliceZeyil;PoliceZeyil[PoliceZeyilKPK])</pre>

<h4>SonDurum_NetPrim (Measure)</h4>
Tahakkuk eden primler toplamından, iade / iptal primler toplamını çıkartır. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyil.md">PoliceZeyil</a>
<pre>SonDurum_NetPrim = SUMX(RELATEDTABLE(PoliceZeyil);PoliceZeyil[PoliceZeyilNetPrim])</pre>

<h4>HasarDosyaAdet (Measure)</h4>
Poliçenin hasar adedini belirtir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/HasarDosya.md">HasarDosya</a>
<pre>HasarDosyaAdet = COUNTROWS(RELATEDTABLE(HasarDosya))</pre>

<h4>HasarMagdurAdet (Measure)</h4>
Poliçenin hasar mağdur adedini belirtir. Trafik hasarlarında her bir hasarda birden fazla mağdur olabilir. Trafik dışı hasarlarda tek mağdur vardır. Bu formül, tüm mağdurların toplamını verir. İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/HasarMagdur.md">HasarMagdur</a>
<pre>HasarMagdurAdet = COUNTROWS(RELATEDTABLE(HasarMagdur))</pre>

<h4>X_TeminatNetPrimToplami (Measure)</h4>
Poliçedeki X kodlu teminatın net prim toplamını verir (zeyiller hesaba katılmış olarak). İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilTeminat.md">PoliceZeyilTeminat</a>
<pre>X_TeminatNetPrimToplami = SUMX(FILTER(RELATEDTABLE(PoliceZeyilTeminat);PoliceZeyilTeminat[TEM_KODU]="X");PoliceZeyilTeminat[TeminatNetPrim])</pre>

<h4>X_Y_TeminatlarNetPrimToplami (Measure)</h4>
Poliçedeki X ve Y kodlu teminatların net prim toplamını verir (zeyiller hesaba katılmış olarak). || (OR) operatörü kullanılarak istenilen sayıda teminat kodu girilebilir İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilTeminat.md">PoliceZeyilTeminat</a>
<pre>X_Y_TeminatlarNetPrimToplami = SUMX(FILTER(RELATEDTABLE(PoliceZeyilTeminat);PoliceZeyilTeminat[TEM_KODU]="X" || PoliceZeyilTeminat[TEM_KODU]="Y");PoliceZeyilTeminat[TeminatNetPrim])</pre>

<h4>X_Haric_TeminatNetPrimToplami (Measure)</h4>
Poliçedeki X kodlu teminat harici tüm teminatların net prim toplamını verir (zeyiller hesaba katılmış olarak). İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilTeminat.md">PoliceZeyilTeminat</a>
<pre>X_Haric_TeminatNetPrimToplami = SUMX(FILTER(RELATEDTABLE(PoliceZeyilTeminat);PoliceZeyilTeminat[TEM_KODU]<>"X");PoliceZeyilTeminat[TeminatNetPrim])</pre>

<h4>X_Y_Haric_TeminatNetPrimToplami (Measure)</h4>
Poliçedeki X ve Y kodlu teminatlar harici tüm teminatların net prim toplamını verir (zeyiller hesaba katılmış olarak). İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilTeminat.md">PoliceZeyilTeminat</a>
<pre>X_Y_Haric_TeminatNetPrimToplami = SUMX(FILTER(RELATEDTABLE(PoliceZeyilTeminat);PoliceZeyilTeminat[TEM_KODU]<>"X" && PoliceZeyilTeminat[TEM_KODU]<>"Y");PoliceZeyilTeminat[TeminatNetPrim])</pre>

<h4>PoliceTeminatNetPrim (Measure)</h4>
Poliçedeki tüm teminatların net prim toplamını verir (zeyiller hesaba katılmış olarak). İhtiyaç duyulan ilişkili tablo <a href="../VeriKaynaklari/PoliceZeyilTeminat.md">PoliceZeyilTeminat</a>
<pre>PoliceTeminatNetPrim = SUMX(RELATEDTABLE(PoliceZeyilTeminat);PoliceZeyilTeminat[TeminatNetPrim])</pre>








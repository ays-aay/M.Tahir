---
title: Kütüphane Analizleri
layout: page-full-width
permalink: /analiz.html
---

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />

<style>
    .analiz-nav {
        display: flex;
        justify-content: center;
        gap: 15px;
        margin: 30px 0 40px;
        flex-wrap: wrap;
    }
    .tab-btn {
        background: white;
        border: 1px solid #e2e8f0;
        color: #1e3a8a;
        padding: 12px 24px;
        border-radius: 8px;
        cursor: pointer;
        font-weight: 600;
        transition: all 0.2s;
    }
    .tab-btn:hover { background: #f1f5f9; }
    .tab-btn.active { background: #2563eb; color: white; border-color: #2563eb; }
    .analiz-section { display: none; }
    .analiz-section.active { display: block; animation: fadeIn 0.4s ease; }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    .analiz-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 25px; }
    @media (max-width: 900px) { .analiz-grid { grid-template-columns: 1fr; } }
    .analiz-card {
        background: #ffffff;
        border: 1px solid #e2e8f0;
        border-radius: 12px;
        padding: 20px;
        box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
    }
    .analiz-card h3 { margin-bottom: 20px; font-size: 1.2rem; color: #1e3a8a; border-left: 4px solid #2563eb; padding-left: 12px; }
    .analiz-card img { max-width: 100%; height: auto; border-radius: 6px; cursor: zoom-in; }
    #map, #heatmap { height: 600px; border-radius: 12px; border: 1px solid #e2e8f0; background: #eee; width: 100%; }
    #analiz-modal {
        display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
        background: rgba(0,0,0,0.9); z-index: 10000; justify-content: center; align-items: center; padding: 20px;
    }
    #analiz-modal img { max-width: 95%; max-height: 95%; border-radius: 8px; cursor: zoom-out; background: white; }
</style>

<div class="analiz-nav">
    <button class="tab-btn active" onclick="switchTab('cities', this)">Şehir Analizi</button>
    <button class="tab-btn" onclick="switchTab('countries', this)">Ülke Analizi</button>
    <button class="tab-btn" onclick="switchTab('libraries', this)">Kütüphane Analizi</button>
    <button class="tab-btn" onclick="switchTab('map-view', this)">Harita</button>
    <button class="tab-btn" onclick="switchTab('heat-view', this)">Yoğunluk (Isı) Haritası</button>
</div>

<div class="container-fluid px-4">
    <div id="cities" class="analiz-section active">
        <div class="analiz-grid">
            <div class="analiz-card"><h3>Şehirlere Göre Eser Dağılımı</h3><img src="{{ '/objects/sehir_bar.png' | relative_url }}" onclick="openZoom(this)" alt="Şehir Bar Grafiği"></div>
            <div class="analiz-card"><h3>Şehir Pasta Grafiği</h3><img src="{{ '/objects/sehir_pie.png' | relative_url }}" onclick="openZoom(this)" alt="Şehir Pasta Grafiği"></div>
            <div class="analiz-card"><h3>En Fazla Esere Sahip Şehirler</h3><img src="{{ '/objects/sehir_top10.png' | relative_url }}" onclick="openZoom(this)" alt="Top 10 Şehir"></div>
            <div class="analiz-card"><h3>Şehir Kümülatif Dağılım</h3><img src="{{ '/objects/sehir_cumulative.png' | relative_url }}" onclick="openZoom(this)" alt="Kümülatif Dağılım"></div>
        </div>
    </div>
    <div id="countries" class="analiz-section">
        <div class="analiz-grid">
            <div class="analiz-card"><h3>Ülkelere Göre Eser Dağılımı</h3><img src="{{ '/objects/ulke_bar.png' | relative_url }}" onclick="openZoom(this)" alt="Ülke Bar Grafiği"></div>
            <div class="analiz-card"><h3>Ülke Pasta Grafiği</h3><img src="{{ '/objects/ulke_pie.png' | relative_url }}" onclick="openZoom(this)" alt="Ülke Pasta Grafiği"></div>
        </div>
    </div>
    <div id="libraries" class="analiz-section">
        <div class="analiz-grid">
            <div class="analiz-card"><h3>Kütüphanelere Göre Eser Dağılımı</h3><img src="{{ '/objects/kutuphane_bar.png' | relative_url }}" onclick="openZoom(this)" alt="Kütüphane Bar Grafiği"></div>
            <div class="analiz-card"><h3>En Fazla Esere Sahip Kütüphaneler</h3><img src="{{ '/objects/kutuphane_top10.png' | relative_url }}" onclick="openZoom(this)" alt="Top 10 Kütüphane"></div>
        </div>
    </div>
    <div id="map-view" class="analiz-section"><div class="analiz-card"><h3>Kütüphane Konumları</h3><div id="map"></div></div></div>
    <div id="heat-view" class="analiz-section"><div class="analiz-card"><h3>Eser Yoğunluğu (Isı Haritası)</h3><div id="heatmap"></div></div></div>
</div>

<div id="analiz-modal" onclick="this.style.display='none'"><img id="analiz-modal-img"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://leaflet.github.io/Leaflet.heat/dist/leaflet-heat.js"></script>
<script>
    const mapData = [{"n": "S\u00fcleymaniye", "lt": 41.0161522086512, "lg": 28.9639603676263, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 316}, {"n": "Beyaz\u0131t", "lt": 41.0112288381767, "lg": 28.9655158313884, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 59}, {"n": "Nuruosmaniye", "lt": 41.0102587667774, "lg": 28.9706631927802, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 63}, {"n": "\u0130stanbul \u00dcniversitesi", "lt": 41.0127181146775, "lg": 28.9618372687258, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 24}, {"n": "Topkap\u0131 Saray\u0131", "lt": 41.0116490030607, "lg": 28.9834110829694, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 25}, {"n": "\u0130stanbul Arkeoloji M\u00fczeleri", "lt": 41.0119364381759, "lg": 28.9812661234469, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 5}, {"n": "T\u00fcrk Tarih Kurumu", "lt": 39.9313984412271, "lg": 32.8560313901933, "c": "Ankara", "u": "T\u00fcrkiye", "v": 40}, {"n": "Edirne Selimiye Yazma Eser", "lt": 41.6801554058122, "lg": 26.5523889331729, "c": "Edirne", "u": "T\u00fcrkiye", "v": 9}, {"n": "Hac\u0131 Selim A\u011fa", "lt": 41.0249357996922, "lg": 29.0170753288358, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 9}, {"n": "K\u00f6pr\u00fcl\u00fc", "lt": 41.0082474903206, "lg": 28.9725273576714, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 20}, {"n": "Murad Molla", "lt": 41.027776675037, "lg": 28.9473413423283, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 12}, {"n": "Rag\u0131b Pa\u015fa", "lt": 41.0091716242054, "lg": 28.9656324576716, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 15}, {"n": "Millet", "lt": 41.0170748506218, "lg": 28.9499245018509, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 18}, {"n": "Manisa Yazma Eser", "lt": 38.6156254746829, "lg": 27.4353247412881, "c": "Manisa", "u": "T\u00fcrkiye", "v": 3}, {"n": "T\u00fcrk ve \u0130slam Eserleri M\u00fczesi", "lt": 41.0063207279663, "lg": 28.9748059271485, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 2}, {"n": "At\u0131f Efendi", "lt": 41.0170778506181, "lg": 28.9600988288358, "c": "\u0130stanbul", "u": "T\u00fcrkiye", "v": 1}, {"n": "Oriental Public Library ", "lt": 25.6194021118221, "lg": 85.1624840199649, "c": "Bankipore", "u": "Hindistan", "v": 1}, {"n": "Kalkutta Buhar K\u00fct\u00fcphanesi ", "lt": 22.5332050368602, "lg": 88.3335810576717, "c": "Kalk\u00fcta", "u": "Hindistan", "v": 1}, {"n": "Kahire Mill\u00ee K\u00fct\u00fcphanesi", "lt": 30.06721968343, "lg": 31.2271369904274, "c": "Kahire", "u": "M\u0131s\u0131r", "v": 126}, {"n": "Cezayir Milli K\u00fct\u00fcphanesi", "lt": 36.7485774796796, "lg": 3.07163869069325, "c": "Cezayir", "u": "Cezayir", "v": 4}, {"n": "Kazan Federal \u00dcniversitesi", "lt": 55.7908010159565, "lg": 49.1216551261875, "c": "Kazan", "u": "Rusya", "v": 1}, {"n": "Berlin Devlet K\u00fct\u00fcphanesi", "lt": 52.5077560047923, "lg": 13.3708612239915, "c": "Berlin", "u": "Almanya", "v": 117}, {"n": "Gotha K\u00fct\u00fcphanesi", "lt": 50.9478847109003, "lg": 10.7093788460301, "c": "Gotha", "u": "Almanya", "v": 37}, {"n": "M\u00fcnih Devlet K\u00fct\u00fcphanesi", "lt": 48.1476146646503, "lg": 11.5803002393623, "c": "M\u00fcnih", "u": "Almanya", "v": 45}, {"n": "Paris Mill\u00ee K\u00fct\u00fcphanesi", "lt": 48.8339332273818, "lg": 2.37579298412067, "c": "Paris", "u": "Fransa", "v": 163}, {"n": "Stokholm Kraliyet K\u00fct\u00fcphanesi", "lt": 59.3381358369098, "lg": 18.072031001596, "c": "Stokholm", "u": "\u0130sve\u00e7", "v": 9}, {"n": "Uppsala \u00dcniversitesi K\u00fct\u00fcphanesi", "lt": 59.8552226554285, "lg": 17.6314371362395, "c": "Uppsala", "u": "\u0130sve\u00e7", "v": 25}, {"n": "Petersburg Umumi K\u00fct\u00fcphanesi (Rusya Milli K\u00fct\u00fcphanesi)", "lt": 59.9334029974936, "lg": 30.3352102785678, "c": "Londra", "u": "Rusya", "v": 17}, {"n": "Roma Vatikan K\u00fct\u00fcphanesi", "lt": 41.9048839898964, "lg": 12.4549978153433, "c": "Roma", "u": "Vatikan", "v": 15}, {"n": "Venedik K\u00fct\u00fcphanesi", "lt": 45.4335493764096, "lg": 12.339401298149, "c": "Londra", "u": "\u0130talya", "v": 3}, {"n": "Cambridge \u00dcniversite K\u00fct\u00fcphanesi ", "lt": 52.2052468752737, "lg": 0.1077233, "c": "Cambridge", "u": "\u0130ngiltere", "v": 19}, {"n": "British Museum", "lt": 51.5195338599772, "lg": -0.126938820491652, "c": "Londra", "u": "\u0130ngiltere", "v": 96}, {"n": "Leipzig Belediye K\u00fct\u00fcphanesi", "lt": 51.3342046470006, "lg": 12.3747494423283, "c": "Leipzig", "u": "Almanya", "v": 4}, {"n": "Viyana Milli K\u00fct\u00fcphanesi", "lt": 48.2063824539942, "lg": 16.3669274865075, "c": "Viyana", "u": "Avusturya", "v": 266}, {"n": "Oxford Bodleian Library", "lt": 51.7543718691353, "lg": -1.25404200002206, "c": "Oxford", "u": "\u0130ngiltere", "v": 30}, {"n": "Floransa Laurentian K\u00fct\u00fcphanesi", "lt": 43.7747763692041, "lg": 11.2546200711641, "c": "Floransa", "u": "\u0130talya", "v": 2}, {"n": "Roma Casanatense K\u00fct\u00fcphanesi", "lt": 41.8988476150099, "lg": 12.4794832865075, "c": "Roma", "u": "\u0130talya", "v": 2}, {"n": "Leiden \u00dcniversitesi K\u00fct\u00fcphanesi", "lt": 52.1574103133869, "lg": 4.48142699999999, "c": "Leiden", "u": "Hollanda", "v": 25}, {"n": "Tresoar \u2013 Friz Tarih ve Edebiyat Merkezi", "lt": 53.2036272995313, "lg": 5.79024428465661, "c": "Leeuwarden", "u": "Hollanda", "v": 1}, {"n": "Amsterdam Halk K\u00fct\u00fcphanesi", "lt": 52.3762283070659, "lg": 4.90835703221171, "c": "Amsterdam", "u": "Hollanda", "v": 1}, {"n": "Dresden Memleket K\u00fct\u00fcphanesi", "lt": 51.0287565539879, "lg": 13.7370277288358, "c": "Dresden", "u": "Almanya", "v": 25}, {"n": "Prens Dietrichstein K\u00fct\u00fcphanesi", "lt": 48.8067576922989, "lg": 16.6364540558207, "c": "Mikulov", "u": "\u00c7ekya", "v": 1}, {"n": "Utrecht \u00dcniversite K\u00fct\u00fcphanesi", "lt": 52.0949904349757, "lg": 5.12356222698491, "c": "Utrecht", "u": "Hollanda", "v": 1}, {"n": "Leipzig \u015eehir K\u00fct\u00fcphanesi", "lt": 51.3342046470006, "lg": 12.3747494423283, "c": "Leipzig", "u": "Almanya", "v": 5}, {"n": "Londra Royal As. Soc.", "lt": 51.5265586745389, "lg": -0.135425244179236, "c": "Londra", "u": "\u0130ngiltere", "v": 15}, {"n": "Manchester \u00dcniversitesi K\u00fct\u00fcphanesi", "lt": 53.4645760548282, "lg": -2.23532624232767, "c": "Manchester", "u": "\u0130ngiltere", "v": 20}, {"n": "Escurial K\u00fct\u00fcphanesi", "lt": 40.4456565340475, "lg": -3.7344189458961, "c": "Madrid", "u": "\u0130spanya", "v": 2}, {"n": "Basel \u00dcniversite K\u00fct\u00fcphanesi", "lt": 47.559834281226, "lg": 7.58086368650754, "c": "Basel", "u": "\u0130svi\u00e7re", "v": 5}, {"n": "Hamburg \u015eehir K\u00fct\u00fcphanesi", "lt": 53.5497847853875, "lg": 10.0085323865075, "c": "Hamburg", "u": "Almanya", "v": 3}, {"n": "Paris Arsenal K\u00fct\u00fcphanesi", "lt": 48.8504834973113, "lg": 2.36360177301508, "c": "Paris", "u": "Fransa", "v": 3}, {"n": "Milan Ambrosiana K\u00fct\u00fcphanesi", "lt": 45.463599496937, "lg": 9.1857666423283, "c": "Milan", "u": "\u0130talya", "v": 4}, {"n": "Halle Francke Vak\u0131flar\u0131 K\u00fct\u00fcphanesi", "lt": 51.4781043415075, "lg": 11.9709711, "c": "Halle", "u": "Almanya", "v": 4}, {"n": "G\u00f6ttingen \u00dcniversite K\u00fct\u00fcphanesi", "lt": 51.5416881100557, "lg": 9.93760592883584, "c": "G\u00f6ttingen", "u": "Almanya", "v": 2}, {"n": "Petersburg Asya M\u00fczesi/\u015eark Enstit\u00fcs\u00fc", "lt": 59.944216783779, "lg": 30.3219009558207, "c": "Petersburg", "u": "Rusya", "v": 34}, {"n": "Leipzig \u00dcniversite K\u00fct\u00fcphanesi", "lt": 51.3323685339937, "lg": 12.3684250595226, "c": "Leipzig", "u": "Almanya", "v": 6}, {"n": "Wolfenb\u00fcttel Memleket K\u00fct\u00fcphanesi", "lt": 52.164383159017, "lg": 10.5302791423283, "c": "Wolfenb\u00fcttel", "u": "Almanya", "v": 1}, {"n": "Amsterdam Bilimler Akademisi", "lt": 52.3714286574962, "lg": 4.89939552698491, "c": "Amsterdam", "u": "Hollanda", "v": 1}, {"n": "Hunterian M\u00fczesi", "lt": 55.8731653333335, "lg": -4.28901497565788, "c": "Glasgow", "u": "\u0130sko\u00e7ya", "v": 2}, {"n": "Torino Milli K\u00fct\u00fcphanesi", "lt": 45.0683947285262, "lg": 7.68689671534338, "c": "Torino", "u": "\u0130talya", "v": 2}, {"n": "Danimarka Kraliyet K\u00fct\u00fcphanesi", "lt": 55.6735083015164, "lg": 12.5826588, "c": "Kopenhag", "u": "Danimarka", "v": 8}, {"n": "Cambridge King's College K\u00fct\u00fcphanesi", "lt": 52.2044999781374, "lg": 0.117349373015084, "c": "Cambridge", "u": "\u0130ngiltere", "v": 1}, {"n": "Breslau K\u00fct\u00fcphanesi", "lt": 51.114024017317, "lg": 17.0365290999999, "c": "Breslau", "u": "Polonya", "v": 1}, {"n": "Petersburg \u00dcniversite K\u00fct\u00fcphanesi", "lt": 59.9429041538172, "lg": 30.2978286129788, "c": "Petersburg", "u": "Rusya", "v": 4}, {"n": "Bolonya \u00dcniversitesi K\u00fct\u00fcphanesi", "lt": 44.4970055494825, "lg": 11.3524093999999, "c": "Bolonya", "u": "\u0130talya", "v": 5}, {"n": "Almanya Turing \u00dcniversitesi K\u00fct\u00fcphanesi", "lt": 50.9303372500121, "lg": 11.5874522682011, "c": "Turing", "u": "Almanya", "v": 1}, {"n": "Erlangen K\u00fct\u00fcphanesi", "lt": 49.5968141483872, "lg": 11.0076744306867, "c": "Erlangen", "u": "Almanya", "v": 1}, {"n": "Petersburg Rumenzof M\u00fczesi ", "lt": 59.9329641627469, "lg": 30.2893823719895, "c": "Petersburg", "u": "Rusya", "v": 1}, {"n": "Graz K\u00fct\u00fcphanesi", "lt": 47.0783998977549, "lg": 15.4503621074388, "c": "Graz", "u": "Avusturya", "v": 1}, {"n": "Serez K\u00fct\u00fcphanesi", "lt": 41.0879742982999, "lg": 23.5508629541376, "c": "Serez", "u": "Yunanistan", "v": 1}, {"n": "Venedik San Marco K\u00fct\u00fcphanesi", "lt": 45.4335192322334, "lg": 12.3394442101988, "c": "Venedik", "u": "\u0130talya", "v": 1}, {"n": "Lund \u00dcniversite K\u00fct\u00fcphanesi", "lt": 55.709020497322, "lg": 13.1972130126896, "c": "Lund", "u": "\u0130sve\u00e7", "v": 1}, {"n": "Macaristan Bilimler Akademisi", "lt": 47.5012429264648, "lg": 19.0464721256632, "c": "Budape\u015fte", "u": "Macaristan", "v": 1}];
    const heatData = [[41.0161522086512, 28.9639603676263, 316], [41.0112288381767, 28.9655158313884, 59], [41.0102587667774, 28.9706631927802, 63], [41.0127181146775, 28.9618372687258, 24], [41.0116490030607, 28.9834110829694, 25], [41.0119364381759, 28.9812661234469, 5], [39.9313984412271, 32.8560313901933, 40], [41.6801554058122, 26.5523889331729, 9], [41.0249357996922, 29.0170753288358, 9], [41.0082474903206, 28.9725273576714, 20], [41.027776675037, 28.9473413423283, 12], [41.0091716242054, 28.9656324576716, 15], [41.0170748506218, 28.9499245018509, 18], [38.6156254746829, 27.4353247412881, 3], [41.0063207279663, 28.9748059271485, 2], [41.0170778506181, 28.9600988288358, 1], [25.6194021118221, 85.1624840199649, 1], [22.5332050368602, 88.3335810576717, 1], [30.06721968343, 31.2271369904274, 126], [36.7485774796796, 3.07163869069325, 4], [55.7908010159565, 49.1216551261875, 1], [52.5077560047923, 13.3708612239915, 117], [50.9478847109003, 10.7093788460301, 37], [48.1476146646503, 11.5803002393623, 45], [48.8339332273818, 2.37579298412067, 163], [59.3381358369098, 18.072031001596, 9], [59.8552226554285, 17.6314371362395, 25], [59.9334029974936, 30.3352102785678, 17], [41.9048839898964, 12.4549978153433, 15], [45.4335493764096, 12.339401298149, 3], [52.2052468752737, 0.1077233, 19], [51.5195338599772, -0.126938820491652, 96], [51.3342046470006, 12.3747494423283, 4], [48.2063824539942, 16.3669274865075, 266], [51.7543718691353, -1.25404200002206, 30], [43.7747763692041, 11.2546200711641, 2], [41.8988476150099, 12.4794832865075, 2], [52.1574103133869, 4.48142699999999, 25], [53.2036272995313, 5.79024428465661, 1], [52.3762283070659, 4.90835703221171, 1], [51.0287565539879, 13.7370277288358, 25], [48.8067576922989, 16.6364540558207, 1], [52.0949904349757, 5.12356222698491, 1], [51.3342046470006, 12.3747494423283, 5], [51.5265586745389, -0.135425244179236, 15], [53.4645760548282, -2.23532624232767, 20], [40.4456565340475, -3.7344189458961, 2], [47.559834281226, 7.58086368650754, 5], [53.5497847853875, 10.0085323865075, 3], [48.8504834973113, 2.36360177301508, 3], [45.463599496937, 9.1857666423283, 4], [51.4781043415075, 11.9709711, 4], [51.5416881100557, 9.93760592883584, 2], [59.944216783779, 30.3219009558207, 34], [51.3323685339937, 12.3684250595226, 6], [52.164383159017, 10.5302791423283, 1], [52.3714286574962, 4.89939552698491, 1], [55.8731653333335, -4.28901497565788, 2], [45.0683947285262, 7.68689671534338, 2], [55.6735083015164, 12.5826588, 8], [52.2044999781374, 0.117349373015084, 1], [51.114024017317, 17.0365290999999, 1], [59.9429041538172, 30.2978286129788, 4], [44.4970055494825, 11.3524093999999, 5], [50.9303372500121, 11.5874522682011, 1], [49.5968141483872, 11.0076744306867, 1], [59.9329641627469, 30.2893823719895, 1], [47.0783998977549, 15.4503621074388, 1], [41.0879742982999, 23.5508629541376, 1], [45.4335192322334, 12.3394442101988, 1], [55.709020497322, 13.1972130126896, 1], [47.5012429264648, 19.0464721256632, 1]];
    const maxC = 1794;
    let maps = { map: null, heat: null };

    function switchTab(id, btn) {
        document.querySelectorAll('.analiz-section').forEach(s => s.classList.remove('active'));
        document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        btn.classList.add('active');
        if (id === 'map-view') initMap();
        if (id === 'heat-view') initHeat();
    }

    function initMap() {
        if (maps.map) return;
        maps.map = L.map('map').setView([41, 29], 4);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(maps.map);
        mapData.forEach(p => {
            L.marker([p.lt, p.lg]).addTo(maps.map)
                .bindPopup(`<b>${p.n}</b><br>${p.c}, ${p.u}<br>Eser: ${p.v}`);
        });
    }

    function initHeat() {
        if (maps.heat) return;
        maps.heat = L.map('heatmap').setView([41, 29], 4);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(maps.heat);
        const intensityPoints = heatData.map(p => {
            let count = p[2];
            let intensity = Math.log(count + 1) / Math.log(maxC + 1);
            return [p[0], p[1], intensity];
        });
        L.heatLayer(intensityPoints, {
            radius: 45,
            blur: 25,
            max: 0.6,
            minOpacity: 0.5,
            gradient: { 0.2: 'blue', 0.4: 'cyan', 0.6: 'lime', 0.8: 'yellow', 1.0: 'red' }
        }).addTo(maps.heat);
    }

    function openZoom(el) {
        document.getElementById('analiz-modal').style.display = 'flex';
        document.getElementById('analiz-modal-img').src = el.src;
    }

    window.addEventListener('resize', () => {
        if(maps.map) maps.map.invalidateSize();
        if(maps.heat) maps.heat.invalidateSize();
    });
</script>

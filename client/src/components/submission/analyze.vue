<!--
Copyright 2017 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <modal id="submission-analyze" :state="state" hideable backdrop
    @hide="$emit('hide')">
    <template #title>{{ $t('title') }}</template>
    <template #body>
      <div id="submission-analyze-head">
        <div class="modal-introduction">
          <i18n-t tag="p" keypath="introduction[0]">
            <template #powerBi>
              <a href="https://powerbi.microsoft.com/en-us/" target="_blank" rel="noopener">Microsoft Power BI</a>
            </template>
            <template #r>
              <a href="https://www.r-project.org" target="_blank" rel="noopener">R</a>
            </template>
          </i18n-t>
          <p>{{ $t('introduction[1]') }}</p>
          <p>{{ $t('introduction[2]') }}</p>
        </div>
        <ul class="nav nav-tabs">
          <li :class="tabClass('microsoft')" role="presentation">
            <a href="#" @click.prevent="setTool('microsoft')">{{ $t('tab.microsoft') }}</a>
          </li>
          <li :class="tabClass('r')" role="presentation">
            <a href="#" @click.prevent="setTool('r')">R</a>
          </li>
          <li :class="tabClass('other')" role="presentation">
            <a href="#" @click.prevent="setTool('other')">{{ $t('tab.other') }}</a>
          </li>
        </ul>
      </div>
      <div id="submission-analyze-odata-url" class="modal-introduction">
        <selectable>{{ odataUrl }}</selectable>
      </div>
      <div id="submission-analyze-tool-help" class="modal-introduction">
        <i18n-t v-if="tool === 'microsoft'" tag="p" keypath="help.microsoft.full">
          <template #pageForExcel>
            <a href="https://support.office.com/en-us/article/connect-to-an-odata-feed-power-query-4441a94d-9392-488a-a6a9-739b6d2ad500" target="_blank" rel="noopener">{{ $t('help.microsoft.pageForExcel') }}</a>
          </template>
          <template #pageForPowerBi>
            <a href="https://docs.microsoft.com/en-us/power-bi/desktop-connect-odata" target="_blank" rel="noopener">{{ $t('help.microsoft.pageForPowerBi') }}</a>
          </template>
        </i18n-t>
        <template v-else-if="tool === 'r'">
          <i18n-t tag="p" keypath="help.r[0].full">
            <template #r>
              <a href="https://www.r-project.org" target="_blank" rel="noopener">R</a>
            </template>
            <template #ruODK>
              <a href="https://docs.ropensci.org/ruODK/index.html" target="_blank" rel="noopener">ruODK</a>
            </template>
            <template #oData>
              <a href="https://docs.ropensci.org/ruODK/articles/odata-api.html" target="_blank" rel="noopener">OData</a>
            </template>
            <template #restful>
              <a href="https://docs.ropensci.org/ruODK/articles/restful-api.html" target="_blank" rel="noopener">{{ $t('help.r[0].restful') }}</a>
            </template>
          </i18n-t>
          <i18n-t tag="p" keypath="help.r[1].full">
            <template #here>
              <a href="https://docs.ropensci.org/ruODK/CONTRIBUTING.html" target="_blank" rel="noopener">{{ $t('help.r[1].here') }}</a>
            </template>
          </i18n-t>
        </template>
        <i18n-t v-else-if="tool === 'other'" tag="p" keypath="help.other.full">
          <template #article>
            <a href="https://odkcentral.docs.apiary.io/#reference/odata-endpoints" target="_blank" rel="noopener">{{ $t('help.other.article') }}</a>
          </template>
        </i18n-t>
      </div>
      <div id="submission-analyze-actions-container">
        <div class="modal-actions">
          <button type="button" class="btn btn-primary" @click="$emit('hide')">
            {{ $t('action.done') }}
          </button>
        </div>
      </div>
    </template>
  </modal>
</template>

<script>
import Modal from '../modal.vue';
import Selectable from '../selectable.vue';

import { apiPaths } from '../../util/request';
import { useRequestData } from '../../request-data';

export default {
  name: 'SubmissionAnalyze',
  components: { Modal, Selectable },
  props: {
    state: Boolean
  },
  emits: ['hide'],
  setup() {
    const { form } = useRequestData();
    return { form };
  },
  data() {
    return {
      tool: 'microsoft'
    };
  },
  computed: {
    odataUrl() {
      if (!this.form.dataExists) return '';
      const path = apiPaths.odataSvc(this.form.projectId, this.form.xmlFormId);
      return `${window.location.origin}${path}`;
    }
  },
  watch: {
    state(state) {
      if (!state) this.tool = 'microsoft';
    }
  },
  methods: {
    tabClass(tool) {
      return { active: this.tool === tool };
    },
    setTool(tool) {
      this.tool = tool;
    }
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

#submission-analyze .modal-body {
  padding-left: 0;
  padding-right: 0;

  .modal-introduction {
    margin-bottom: 10px;
  }

  #submission-analyze-head {
    border-bottom: 1px solid $color-subpanel-border-strong;
    padding-left: $padding-modal-body;
    padding-right: $padding-modal-body;

    .nav-tabs {
      margin-top: 15px;
    }
  }

  #submission-analyze-odata-url {
    background-color: $color-subpanel-background;
    margin-bottom: 10px;
    padding: 12px $padding-modal-body;
  }

  #submission-analyze-tool-help {
    margin-top: 15px;
    padding-left: $padding-modal-body;
    padding-right: $padding-modal-body;

    &:empty {
      margin-top: 0px;
    }
  }

  #submission-analyze-actions-container {
    padding-left: $padding-modal-body;
    padding-right: $padding-modal-body;
  }
}
</style>

<i18n lang="json5">
{
  "en": {
    // This is the title at the top of a pop-up.
    "title": "Using OData",
    "introduction": [
      // The text of {powerBi} is "Microsoft Power BI". The text of {r} is "R".
      // {powerBi} and {r} are both links.
      "OData is a new standard for transferring data between tools and services. Free and powerful analysis tools like Excel, {powerBi}, and {r} can fetch data over OData for analysis.",
      "There are many advantages to OData, but importantly it supports the full-fidelity transfer of complicated types like numbers and geographic data, and it enables the latest version of your data to synchronize easily with any tools using it.",
      "To begin using OData, select your tool and copy the link into it."
    ],
    "tab": {
      "microsoft": "Excel/Power BI",
      // This is the text of a navigation tab. "Other" refers to "other tool".
      "other": "Other"
    },
    "help": {
      "microsoft": {
        "full": "For help using OData with Excel, see {pageForExcel}. For help with Power BI, see {pageForPowerBi}.",
        "pageForExcel": "this page",
        "pageForPowerBi": "this page"
      },
      "r": [
        {
          // The text of {r} is "R". The text of {ruODK} is "ruODK". The text of
          // {oData} is "OData". All three are links.
          "full": "To access Central data from {r}, we recommend you use {ruODK}. See ruODK???s vignettes for examples of using both the {oData} and the {restful} API.",
          "restful": "RESTful"
        },
        {
          "full": "Just like ODK itself, ruODK is developed and supported by community members. If you wish to help improve it, you can find information {here}.",
          "here": "here"
        }
      ],
      "other": {
        "full": "For a full description of our OData support, please see {article}.",
        "article": "this article"
      }
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "Pomoc?? OData",
    "introduction": [
      "OData je nov?? standard pro p??enos dat mezi n??stroji a slu??bami. Bezplatn?? a v??konn?? analytick?? n??stroje, jako je Excel, {powerBi} a {r} mohou na????st data p??es OData k anal??ze.",
      "Existuje mnoho v??hod pro OData, ale d??le??it?? je, ??e podporuje pln?? v??rn?? p??enos komplikovan??ch typ??, jako jsou ????sla a geografick?? data, a umo????uje nejnov??j???? verzi va??ich dat snadno synchronizovat s jak??mikoli n??stroji, kter?? ji pou????vaj??.",
      "Chcete-li za????t pou????vat OData, vyberte sv??j n??stroj a zkop??rujte do n??j odkaz."
    ],
    "tab": {
      "microsoft": "Excel/Power BI",
      "other": "Jin??"
    },
    "help": {
      "microsoft": {
        "full": "N??pov??du k pou????v??n?? OData s Excelem najdete na {pageForExcel}. N??pov??da k Power BI je na {pageForPowerBi}.",
        "pageForExcel": "t??to stran??",
        "pageForPowerBi": "t??to stran??"
      },
      "r": [
        {
          "full": "Pro p????stup k Centr??ln??m dat??m z {r} , doporu??ujeme pou????t {ruODK}. P????klady pou??it?? rozhran?? {oData} API i {restful} API naleznete u vin??ty ruODK.",
          "restful": "RESTful"
        },
        {
          "full": "Stejn?? jako ODK samotn?? je i ruODK vyv??jen a podporov??n ??leny komunity. Chcete-li ji vylep??it, m????ete naj??t informace {here}.",
          "here": "zde"
        }
      ],
      "other": {
        "full": "??pln?? popis podpory OData naleznete v {article}.",
        "article": "tomto ??l??nku"
      }
    }
  },
  "de": {
    "title": "OData verwenden",
    "introduction": [
      "OData ist ein neuer Standard, um Daten zwischen Tools und Diensten zu ??bertragen. Kostenlose und m??chtige Tools wie Excel, {powerBi} und {r} k??nnen Daten f??r Analysen ??ber OData holen.",
      "OData hat viele Vorteile, aber am wichtigsten ist, dass es den wiedergabetreuen Transfer von komplizierten Typen wie Nummern und geografischen Daten unterst??tzt und es erm??glicht die einfache Synchronisation der aktuellste Version Ihrer Daten mit allen Tools, die OData verwenden.",
      "Um OData zu verwenden, w??hlen Sie Ihr Tool und kopieren Sie den Link."
    ],
    "tab": {
      "microsoft": "Excel/Power BI",
      "other": "Anderes"
    },
    "help": {
      "microsoft": {
        "full": "F??r Hilfe zur Verwendung von OData mit Excel, lesen Sie {pageForExcel}. F??r Hilfe zu Power BI, lesen Sie {pageForPowerBi}.",
        "pageForExcel": "diese Seite",
        "pageForPowerBi": "diese Seite"
      },
      "r": [
        {
          "full": "Um auf Central Daten von {r} aus zuzugreifen, empfehlen wir {ruODK} zu benutzen. Schauen Sie ruODKs Vignettes an f??r Beispiele zur Verwendung der {oData} und der {restful} API.",
          "restful": "RESTful"
        },
        {
          "full": "Wie ODK selbst, wird ruODK von Community-Mitgliedern entwickelt und supported. Wenn Sie helfen wollen es zu verbessern, finden Sie Informationen {here}.",
          "here": "hier"
        }
      ],
      "other": {
        "full": "F??r eine ausf??hrliche Beschreibung unserer OData-Unterst??tzung, bitte lesen Sie {article}.",
        "article": "diesen Artikel"
      }
    }
  },
  "es": {
    "title": "Usando protocolo de datos abiertos \"OData\"",
    "introduction": [
      "El protocolo de datos abiertos \"OData\" es un nuevo protocolo que permite realizar transferencia de datos entre aplicaciones y servidores. Herramientas gratuitas y potentes como Excel, {powerBi}, y {r} pueden obtener y analizar datos a trav??s de OData.",
      "Los protocolos de datos abiertos proveen m??ltiples ventajas, la m??s importante es la transferencia fidedigna de informaci??n compleja como n??meros e informaci??n geogr??fica, y la sincronizaci??n sencilla de informaci??n en cualquier herramienta.",
      "Para comenzar a usar un protocolo de datos abiertos, \"OData\" seleccione su herramienta y copie el enlace."
    ],
    "tab": {
      "microsoft": "Excel/ Power BI",
      "other": "Otro"
    },
    "help": {
      "microsoft": {
        "full": "Para mayor informaci??n del uso de OData con Excel, v??ase {pageForExcel}; con Power BI veas?? {pageForPowerBi}.",
        "pageForExcel": "esta p??gina",
        "pageForPowerBi": "esta p??gina"
      },
      "r": [
        {
          "full": "Para acceder a los datos de Central desde {r}, recomendamos usar {ruODK}. Vea las vi??etas de ruODK para leer ejemplos de uso de {oData} y {restful} API.",
          "restful": "RESTful"
        },
        {
          "full": "Tal como ODK, ruODK es desarrollado y apoyado por miembros de la comunidad. Si desea ayudar a mejorarlo, puede encontrar informaci??n {here}.",
          "here": "aqu??"
        }
      ],
      "other": {
        "full": "Para una descripci??n detallada del soporte realizado para OData, l??a {article}.",
        "article": "este art??culo"
      }
    }
  },
  "fr": {
    "title": "Utilisation d'OData",
    "introduction": [
      "OData est une nouvelle norme pour le transfert de donn??es entre outils et services. Des outils d'analyse gratuits et puissants comme Excel, {powerBi} et {r} peuvent r??cup??rer des donn??es par le biais d'OData pour les analyser.",
      "OData pr??sente de nombreux avantages, mais surtout il permet le transfert en toute fid??lit?? de types complexes comme les nombres et les donn??es g??ographiques, et il permet ?? la derni??re version de vos donn??es de se synchroniser facilement avec tous les outils qui l'utilisent.",
      "Pour commencer ?? utiliser OData, s??lectionnez votre outil et y copiez le lien."
    ],
    "tab": {
      "microsoft": "Excel/Power BI",
      "other": "Autre"
    },
    "help": {
      "microsoft": {
        "full": "Pour obtenir de l'aide sur l'utilisation d'OData avec Excel, voir {pageForExcel}. Pour obtenir de l'aide sur Power BI, voir {pageForPowerBi}.",
        "pageForExcel": "cette page",
        "pageForPowerBi": "cette page"
      },
      "r": [
        {
          "full": "Pour acc??der aux donn??es de Central depuis {r}, nous vous recommandons d'utiliser {ruODK}. Voir les vignettes de ruODK pour des exemples d'utilisation des API {oData} et {restful}.",
          "restful": "RESTful"
        },
        {
          "full": "Tout comme ODK, ruODK est d??velopp?? et soutenu par les membres de la communaut??. Si vous souhaitez contribuer ?? son am??lioration, vous pouvez trouver des informations {here}.",
          "here": "ici"
        }
      ],
      "other": {
        "full": "Pour une description compl??te de notre int??gration OData, veuillez consulter {article}.",
        "article": "cet article"
      }
    }
  },
  "id": {
    "title": "Menggunakan OData",
    "introduction": [
      "OData adalah standar baru untuk mentransfer data antara perangkat dan servis. Perangkat analisis gratis dan unggul seperti Excel, {powerBi}, dan {r} dapat mengambil data lewat OData untuk analisis.",
      "Ada banyak keuntungan menggunakan OData. Yang paling penting adalah OData mendukung ketepatan sempurna pemindahan data kompleks seperti angka dan data geografis, dan memudahkan sinkronisasi data terakhir Anda dengan perangkat yang menggunakannya.",
      "Untuk memulai menggunakan OData, pilih perangkat Anda dan salin tautannya ke sana."
    ],
    "tab": {
      "microsoft": "Excel/Power BI",
      "other": "Lainnya"
    },
    "help": {
      "microsoft": {
        "full": "Untuk bantuan penggunaan OData dengan Excel, lihat {pageForExcel}. Untuk bantuan penggunaan Power BI, lihat {pageForPowerBi}.",
        "pageForExcel": "laman ini",
        "pageForPowerBi": "laman ini"
      },
      "r": [
        {
          "full": "Untuk mengakses data Central dari {r}, kami merekomendasikan Anda untuk menggunakan {ruODK}. Lihat skema ruODK untuk contoh penggunaan {oData} dan {restful} API.",
          "restful": "RESTful"
        },
        {
          "full": "Sama seperti ODK, ruODK dikembangkan dan didukung oleh anggota komunitas. Apabila Anda ingin membantu perkembangannya, Anda dapat menemukan informasi lebih lanjut {here}.",
          "here": "di sini"
        }
      ],
      "other": {
        "full": "Untuk deskripsi lengkap tentang dukungan OData kami, silakan lihat {article}.",
        "article": "artikel ini"
      }
    }
  },
  "it": {
    "title": "Utilizzando OData",
    "introduction": [
      "OData ?? un nuovo standard per il trasferimento di dati tra strumenti e servizi. Strumenti di analisi gratuiti e potenti come Excel, {powerBi} e {r} possono recuperare i dati su OData per l'analisi.",
      "Ci sono molti vantaggi nell'utilizzo di OData, ma soprattutto OData supporta il trasferimento completo di dati complicati come numeri e dati geografici e consente all'ultima versione dei tuoi dati di sincronizzarsi facilmente con qualsiasi strumento che lo utilizza.",
      "Per iniziare a utilizzare OData, seleziona il tuo strumento e copia il link al suo interno."
    ],
    "tab": {
      "microsoft": "Excel/Power BI",
      "other": "Altro"
    },
    "help": {
      "microsoft": {
        "full": "Per informazioni sull'utilizzo di OData con Excel, vedere {pageForExcel}. Per assistenza con Power BI, vedere {pageForPowerBi}.",
        "pageForExcel": "questa pagina",
        "pageForPowerBi": "questa pagina"
      },
      "r": [
        {
          "full": "Per accedere ai dati di Central da {r}, ti consigliamo di utilizzare {ruODK}. Vedi le vignette di ruODK per esempi di utilizzo sia di {oData} che {restful} API.",
          "restful": "RESTful"
        },
        {
          "full": "Proprio come ODK stesso, ruODK ?? sviluppato e supportato dai membri della comunit??. Se desideri contribuire a migliorarlo, puoi trovare informazioni {here}.",
          "here": "qui"
        }
      ],
      "other": {
        "full": "Per una descrizione completa del nostro supporto OData, vedere {article}.",
        "article": "questo articolo"
      }
    }
  },
  "ja": {
    "title": "OData?????????",
    "introduction": [
      "OData???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????{powerBi}???{r}???OData????????????????????????????????????????????????????????????",
      "OData??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????OData?????????????????????????????????????????????????????????????????????????????????",
      "ODdata?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    ],
    "tab": {
      "microsoft": "Excel/Power BI",
      "other": "?????????"
    },
    "help": {
      "microsoft": {
        "full": "???????????????OData??????????????????{pageForExcel}?????????????????????????????????Power BI??????????????????{pageForPowerBi}???????????????????????????",
        "pageForExcel": "???????????????",
        "pageForPowerBi": "???????????????"
      },
      "r": [
        {
          "full": "{r}??????Central??????????????????????????????????????????{ruODK}????????????????????????????????????ruODK???????????????????????????{oData}???{restful}API????????????????????????????????????????????????",
          "restful": "RESTful"
        },
        {
          "full": "ODK???????????????ruODK????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????{here}?????????",
          "here": "?????????"
        }
      ],
      "other": {
        "full": "OData???????????????????????????????????????{article}???????????????????????????",
        "article": "?????????"
      }
    }
  },
  "sw": {
    "title": "Kwa kutumia OData",
    "introduction": [
      "OData ni kiwango kipya cha kuhamisha data kati ya zana na huduma. Zana za uchanganuzi zisizolipishwa na zenye nguvu kama vile Excel, {powerBi} na {r} zinaweza kuleta data kupitia OData kwa uchanganuzi.",
      "Kuna faida nyingi kwa OData, lakini muhimu zaidi inaauni uhamishaji wa uaminifu kamili wa aina ngumu kama vile nambari na data ya kijiografia, na huwezesha toleo jipya zaidi la data yako kusawazisha kwa urahisi na zana zozote zinazoitumia.",
      "Ili kuanza kutumia OData, chagua zana yako na unakili kiungo ndani yake."
    ],
    "tab": {
      "microsoft": "Excel/Power BI",
      "other": "Nyingine"
    },
    "help": {
      "microsoft": {
        "full": "Kwa usaidizi wa kutumia OData na Excel, angalia {pageForExcel}. Kwa usaidizi wa Power BI, angalia {pageForPowerBi}.",
        "pageForExcel": "ukurasa huu",
        "pageForPowerBi": "ukurasa huu"
      },
      "r": [
        {
          "full": "Ili kufikia data ya Kati kutoka kwa {r}, tunapendekeza utumie {ruODK}. Tazama vignette za ruODK kwa mifano ya kutumia API ya {oData} na {restful}.",
          "restful": "Tulivu"
        },
        {
          "full": "kama vile ODK yenyewe, ruODK inatengenezwa na kuungwa mkono na wanajamii. Ikiwa ungependa kusaidia kuiboresha, unaweza kupata maelezo {here}.",
          "here": "Hapa"
        }
      ],
      "other": {
        "full": "Kwa maelezo kamili ya usaidizi wetu wa OData, tafadhali angalia {article}.",
        "article": "Makala hii"
      }
    }
  }
}
</i18n>

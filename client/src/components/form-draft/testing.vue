<!--
Copyright 2020 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <div>
    <div id="form-draft-testing-info" class="row">
      <div class="col-xs-8">
        <page-section>
          <template #heading>
            <span>{{ $t('title') }}</span>
            <enketo-fill v-if="formDraft.dataExists" :form-version="formDraft">
              <span class="icon-plus-circle"></span>{{ $t('action.createSubmission') }}
            </enketo-fill>
          </template>
          <template #body>
            <div v-if="formDraft.entityRelated" class="panel-dialog">
              <div class="panel-heading">
                <span class="panel-title">
                  <span class="icon-database"></span>
                  {{ $t('common.datasetsPreview') }}
                </span>
              </div>
              <div class="panel-body">
                <p>
                  {{ $t('datasetsPreview.body[0]') }}
                </p>
                <p>
                  {{ $t('datasetsPreview.body[1]') }}
                </p>
                <p>
                  {{ $t('datasetsPreview.body[2]') }}
                </p>
              </div>
            </div>
            <p>{{ $t('body[0]') }}</p>
            <p>
              <span>{{ $t('body[1]') }}</span>
              <sentence-separator/>
              <i18n-t keypath="moreInfo.helpArticle.full">
                <template #helpArticle>
                  <doc-link to="central-forms/#working-with-form-drafts">{{ $t('moreInfo.helpArticle.helpArticle') }}</doc-link>
                </template>
              </i18n-t>
            </p>
          </template>
        </page-section>
      </div>
      <div class="col-xs-4">
        <float-row>
          <collect-qr v-if="formDraft.dataExists" :settings="qrSettings"
            error-correction-level="Q" :cell-size="3"/>
        </float-row>
      </div>
    </div>

    <loading :state="keys.initiallyLoading"/>
    <submission-list v-show="keys.dataExists" :project-id="projectId"
      :xml-form-id="xmlFormId" draft/>
  </div>
</template>

<script>
// Import PageSection before FloatRow in order to have the same import order as
// FormSubmissions: see https://github.com/vuejs/vue-cli/issues/3771
import PageSection from '../page/section.vue';
import FloatRow from '../float-row.vue';
import CollectQr from '../collect-qr.vue';
import DocLink from '../doc-link.vue';
import EnketoFill from '../enketo/fill.vue';
import Loading from '../loading.vue';
import SentenceSeparator from '../sentence-separator.vue';
import SubmissionList from '../submission/list.vue';

import { apiPaths } from '../../util/request';
import { noop } from '../../util/util';
import { useRequestData } from '../../request-data';

export default {
  name: 'FormDraftTesting',
  components: {
    PageSection,
    FloatRow,
    CollectQr,
    DocLink,
    EnketoFill,
    Loading,
    SentenceSeparator,
    SubmissionList
  },
  props: {
    projectId: {
      type: String,
      required: true
    },
    xmlFormId: {
      type: String,
      required: true
    }
  },
  setup() {
    const { resourceView, createResource } = useRequestData();
    const formDraft = resourceView('formDraft', (data) => data.get());
    const keys = createResource('keys');
    return { formDraft, keys };
  },
  computed: {
    qrSettings() {
      const url = apiPaths.serverUrlForFormDraft(
        this.formDraft.draftToken,
        this.projectId,
        this.xmlFormId
      );
      return {
        general: {
          server_url: `${window.location.origin}${url}`,
          form_update_mode: 'match_exactly',
          autosend: 'wifi_and_cellular'
        },
        project: {
          name: this.$t('collectProjectName', { name: this.formDraft.nameOrId }),
          icon: '????'
        },
        // Collect requires the settings to have an `admin` property.
        admin: {}
      };
    }
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      // We do not reconcile this.keys and this.formDraft.keyId.
      this.keys.request({
        url: apiPaths.submissionKeys(this.projectId, this.xmlFormId, true)
      }).catch(noop);
    }
  }
};
</script>

<style lang="scss">
#form-draft-testing-info {
  .page-section, .float-row {
    margin-bottom: 25px;
  }
}
</style>

<i18n lang="json5">
{
  "en": {
    // This is a title shown above a section of the page.
    "title": "Draft Testing",
    "body": [
      "You can use the configuration code to the right to set up a mobile device to download this Draft. You can also click the New button above to create a new Submission from your web browser.",
      "Draft Submissions go into the test table below, where you can preview and download them. When you publish this Draft Form, its test Submissions will be permanently removed."
    ],
    // This text will be shown in ODK Collect when testing a Draft Form. {name}
    // is the title of the Draft Form.
    "collectProjectName": "[Draft] {name}",
    "datasetsPreview": {
      "body": [
        "This Form can add Entities to a Dataset.",
        "In this preview release of Datasets, you must publish your Form to see the Dataset update. In a future release of Central, you will be able to test Dataset functionality in a Draft state.",
        "For now, we recommend publishing your Dataset-related Forms, and not approving any submissions (and therefore not create any unwanted entities) until you are sure the Form is ready."
      ]
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "Testov??n?? konceptu",
    "body": [
      "Pomoc?? konfigura??n??ho k??du vpravo m????ete nastavit mobiln?? za????zen?? ke sta??en?? tohoto konceptu. M????ete tak?? klepnout na tla????tko Nov?? v????e a vytvo??it nov?? p????sp??vek z webov??ho prohl????e??e.",
      "Pro koncept p????sp??vku p??ejd??te do n????e uveden?? testovac?? tabulky, kde si ho m????ete prohl??dnout a st??hnout. P??i publikov??n?? tohoto konceptu formul????e budou jeho testovac?? p????sp??vky trvale odstran??ny."
    ],
    "collectProjectName": "[N??vrh] {name}",
    "datasetsPreview": {
      "body": [
        "Tento formul???? m????e p??id??vat entity do datov?? sady.",
        "V t??to p??edb????n?? verzi datov??ch sad je nutn?? formul???? publikovat, aby se aktualizace datov?? sady zobrazila. V n??kter?? z budouc??ch verz?? aplikace Central budete moci testovat funkce Datasetu ve stavu n??vrhu.",
        "Prozat??m doporu??ujeme zve??ej??ovat formul????e souvisej??c?? s datovou sadou a neschvalovat ????dn?? pod??n?? (a tedy nevytv????et ????dn?? ne????douc?? entity), dokud si nebudete jisti, ??e je formul???? p??ipraven."
      ]
    }
  },
  "de": {
    "title": "Entwurfs-Test",
    "body": [
      "Sie k??nnen den Konfigurations-Code rechts benutzen, damit Ihr Mobilger??t diesen Entwurf herunterladen kann. Sie k??nnen auch die Schaltfl??che \"Neu\" oben klicken, um eine neue ??bermittlung mit Ihrem Browser zu erstellen.",
      "Entwurfs-??bermittlungen werden in der Testtabelle unten dargestellt. Sie k??nnen Sie dort ansehen und herunterladen. Wenn Sie dieses Entwurfsformular ver??ffentlichen, werden die Test-??bermittlungen irreversibel entfernt."
    ],
    "collectProjectName": "[Entwurf] {name}"
  },
  "es": {
    "title": "Prueba de borrador",
    "body": [
      "Puede usar el c??digo de configuraci??n de la derecha para configurar un dispositivo m??vil y descargar este borrador. Tambi??n puede dar clic al bot??n Nuevo de abajo para crear un nuevo env??o desde el navegador web.",
      "El borrador de los env??os van a la tabla de prueba a continuaci??n, donde puede obtener una vista previa y descargar. Cuando publique este borrador de Formulario, sus env??os de prueba se eliminar??n permanentemente."
    ],
    "collectProjectName": "borrador {name}"
  },
  "fr": {
    "title": "Test de l'??bauche",
    "body": [
      "Vous pouvez utiliser le code de configuration ?? droite pour configurer un appareil mobile afin de t??l??charger cette ??bauche. Vous pouvez aussi cliquer le bouton Nouveau ci-dessus pour cr??er une nouvelle soumission depuis votre navigateur.",
      "Les soumissions de test vont dans le tableau ci-dessous o?? vous pouvez les pr??visualiser et t??l??charger. Quand vous publierez cette ??bauche, ses soumissions de test seront d??finitivement supprim??es."
    ],
    "collectProjectName": "[Brouillon] {name}",
    "datasetsPreview": {
      "body": [
        "Ce formulaire peut ajouter des entit??s ?? un Dataset."
      ]
    }
  },
  "id": {
    "title": "Pengujian Draf",
    "body": [
      "Anda bisa menggunakan kode konfigurasi di sebelah kanan untuk mengatur perangkat seluler untuk mengunduh draf ini. Anda juga bisa mengklik tombol \"Baru\" untuk membuat kiriman data baru lewat web browser Anda.",
      "Draf kiriman data akan mucnul pada tabel tes di bawah, di mana Anda juga bisa melihat pratinjau dan mengunduhnya. Ketika formulir draf diterbitkan, tes kiriman data akan dihapus secara permanen."
    ],
    "collectProjectName": "[Draf]{name}"
  },
  "it": {
    "title": "Testando la bozza",
    "body": [
      "Puoi utilizzare il codice di configurazione a destra per configurare un dispositivo mobile per scaricare questa bozza. Puoi anche fare clic sul pulsante Nuovo sopra per creare un nuovo invio dal tuo browser web.",
      "Le bozze inviate vanno nella tabella di prova sottostante, dove puoi visualizzarle in anteprima e scaricarle. Quando pubblichi questa bozza di formulario, i suoi Invii di prova verranno rimossi in modo permanente."
    ],
    "collectProjectName": "[Draft] {name}"
  },
  "ja": {
    "title": "?????????????????????",
    "body": [
      "????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????Web??????????????????????????????????????????????????????????????????",
      "???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    ],
    "collectProjectName": "[?????????] {name}"
  },
  "sw": {
    "title": "Mtihani wa Rasimu",
    "body": [
      "Unaweza kutumia msimbo wa usanidi ulio kulia ili kusanidi kifaa cha mkononi ili kupakua Rasimu hii. Unaweza pia kubofya kitufe kipya hapo juu ili kuunda Wasilisho jipya kutoka kwa kivinjari chako cha wavuti.",
      "Rasimu ya Mawasilisho huenda kwenye jedwali la majaribio hapa chini, ambapo unaweza kuhakiki na kupakua. Unapochapisha Rasimu ya Fomu hii, Mawasilisho yake ya majaribio yataondolewa kabisa."
    ],
    "collectProjectName": "[Rasimu] {name}"
  }
}
</i18n>

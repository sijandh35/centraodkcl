<!--
Copyright 2021 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <modal id="analytics-preview" :state="state" hideable backdrop large @hide="$emit('hide')">
    <template #title>{{ $t('title') }}</template>
    <template #body>
      <div class="modal-introduction">
        <p>{{ $t('introduction[0]') }}</p>
        <p>{{ $t('introduction[1]') }}</p>
        <p>{{ $t('introduction[2]') }}</p>
      </div>
      <loading :state="analyticsPreview.initiallyLoading"/>
      <template v-if="analyticsPreview.dataExists">
        <analytics-metrics-table :title="$t('common.system')" :metrics="systemSummary"/>
        <div id="analytics-preview-project-summary">
          <span class="header">{{ $t('projects.title') }}</span>
          <span class="explanation">{{ $tcn('projects.subtitle', numProjects) }}</span>
        </div>
        <div id="analytics-preview-project-tables">
          <div id="users-forms-column">
            <analytics-metrics-table :title="$t('resource.users')" :metrics="userSummary"/>
            <analytics-metrics-table :title="$t('resource.forms')" :metrics="formSummary"/>
          </div>
          <div id="submissions-column">
            <analytics-metrics-table :title="$t('resource.submissions')"
              :metrics="submissionSummary"/>
            <analytics-metrics-table :title="$t('submissionStates')"
              :metrics="submissionStateSummary"/>
            <analytics-metrics-table :title="$t('other')"
              :metrics="otherSummary"/>
          </div>
        </div>
        <template v-if="numDatasets > 0">
          <div id="analytics-preview-dataset-summary">
            <span class="header">{{ $t('datasets.title') }}</span>
            <span class="explanation">{{ $tcn('datasets.subtitle', numDatasets) }}</span>
          </div>
          <analytics-metrics-table :title="$t('resource.datasets')"
            :metrics="firstDataset"/>
        </template>
      </template>
      <div class="modal-actions">
        <button type="button" class="btn btn-primary" @click="$emit('hide')">
          {{ $t('action.close') }}
        </button>
      </div>
    </template>
  </modal>
</template>

<script>
import { omit, pick, flatten } from 'ramda';

import Loading from '../loading.vue';
import Modal from '../modal.vue';
import AnalyticsMetricsTable from './metrics-table.vue';

import { noop } from '../../util/util';
import { useRequestData } from '../../request-data';

// Metrics to filter out (with pick/omit) and put in
// a separate table
const submissionStateMetrics = [
  'num_submissions_received',
  'num_submissions_approved',
  'num_submissions_has_issues',
  'num_submissions_rejected',
  'num_submissions_edited'
];

export default {
  name: 'AnalyticsPreview',
  components: { AnalyticsMetricsTable, Loading, Modal },
  props: {
    state: Boolean
  },
  emits: ['hide'],
  setup() {
    const { createResource } = useRequestData();
    const analyticsPreview = createResource('analyticsPreview');
    return { analyticsPreview };
  },
  computed: {
    systemSummary() {
      return this.analyticsPreview.system;
    },
    firstProject() {
      // eslint-disable-next-line arrow-body-style
      return this.analyticsPreview.projects.reduce((a, b) => {
        return (a.submissions.num_submissions_received.recent > b.submissions.num_submissions_received.recent) ? a : b;
      });
    },
    firstDataset() {
      const { id, ...ds } = flatten(this.analyticsPreview.projects.map(p => p.datasets)).reduce((a, b) => ((a.num_entities.recent > b.num_entities.recent) ? a : b), { num_entities: {} });
      return ds;
    },
    numDatasets() {
      return flatten(this.analyticsPreview.projects.map(p => p.datasets)).length;
    },
    userSummary() {
      return this.firstProject.users;
    },
    formSummary() {
      return this.firstProject.forms;
    },
    submissionSummary() {
      // The submission metrics come from the API together but will be split.
      // This summary is for submission metrics NOT about state.
      return omit(submissionStateMetrics, this.firstProject.submissions);
    },
    submissionStateSummary() {
      // The submission metrics come from the API together but will be split.
      // This summary is for submission state metrics (approved, rejected, etc.)
      return pick(submissionStateMetrics, this.firstProject.submissions);
    },
    numProjects() {
      return this.analyticsPreview.projects.length;
    },
    otherSummary() {
      return this.firstProject.other;
    }
  },
  watch: {
    state(state) {
      if (state)
        this.fetchData();
      else
        this.analyticsPreview.reset();
    }
  },
  methods: {
    fetchData() {
      this.analyticsPreview.request({ url: '/v1/analytics/preview' })
        .catch(noop);
    }
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

#analytics-preview {
  .modal-introduction > p {
    max-width: 100%;
  }
}

#analytics-preview-project-summary, #analytics-preview-dataset-summary {
  padding-bottom: 5px;

  .header {
    font-weight: 500;
    padding-right: 10px;
  }
}

#analytics-preview-project-tables {
  display: flex;
  flex-wrap: wrap;

  #users-forms-column, #submissions-column {
    margin-right: 5px;
    margin-left: 5px;
    flex-grow: 1;
  }

  margin-left: -5px;
  margin-right: -5px;
}
</style>

<i18n lang="json5">
{
  "en": {
    // This is the title at the top of a pop-up.
    "title": "Anonymized Usage Report",
    "introduction": [
      "Thank you for thinking about sending some usage information. This data will help us prioritize your needs!",
      "Shown here is the report we are collecting currently. To respond to new features and needs, we will sometimes change what is reported, but we will only ever gather summary averages like you see here.",
      "You can always come here to see what is being collected."
    ],
    "projects": {
      // This is the title shown above a series of metrics about Project usage.
      "title": "Project Summaries",
      "subtitle": "(Showing the most active Project of {count} Project) | (Showing the most active Project of {count} Projects)"
    },
    // This is the title of a single table in the analytics metrics report
    // of metrics about submission state (approved, rejected, etc)
    "submissionStates": "Submission States",
    "datasets": {
      // This is the title shown above a series of metrics about Datasets and Entities usage.
      "title": "Dataset and Entity Summaries",
      "subtitle": "(Showing the most active Dataset of {count} Dataset) | (Showing the most active Dataset of {count} Datasets)"
    },
    // This is the title of a single table in the analytics metrics report
    // of other additional metrics that don't fit into other categories
    "other": "Other"
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "Anonymizovan?? zpr??va o pou??it??",
    "introduction": [
      "D??kujeme, ??e uva??ujete o zasl??n?? informac?? o pou????v??n??. Tyto ??daje n??m pomohou stanovit priority va??ich pot??eb!",
      "Zde je zobrazena zpr??va, kterou v sou??asn?? dob?? shroma????ujeme. Abychom mohli reagovat na nov?? funkce a pot??eby, budeme ob??as m??nit to, co se hl??s??, ale v??dy budeme shroma????ovat pouze souhrnn?? pr??m??ry, kter?? vid??te zde.",
      "V??dy se sem m????ete p??ij??t pod??vat, co se sb??r??."
    ],
    "projects": {
      "title": "Shrnut?? projektu",
      "subtitle": "(Zobrazuje {count} nejaktivn??j???? projekt projektu) | (Zobrazuje {count} nejaktivn??j???? projekty projektu) | (Zobrazuje {count} nejaktivn??j????ch projekt?? projektu) | (Zobrazuje {count} nejaktivn??j????ch projekt?? projektu)"
    },
    "submissionStates": "Stavy odesl??n??",
    "other": "Jin??"
  },
  "de": {
    "title": "Anonymisierter Nutzungsbericht",
    "introduction": [
      "Dank, dass du dar??ber nachdenkst uns Nutzungsinformationen zu senden. Diese Daten helfen uns, eure Bed??rfnisse zu priorisieren!",
      "Hier ersichtlich ist der Bericht zur aktuellen Datensammlung. Um auf neue Funktionen und Anforderungen zu reagieren, ??ndern wir manchmal die Berichte, aber wir sammeln immer nur Durchschnittswerte, wie hier gezeigt.",
      "Du kannst jederzeit hierher kommen, um zu sehen, welche Daten gesammelt werden."
    ],
    "projects": {
      "title": "Projekt-??bersicht",
      "subtitle": "(Zeigt das aktivste Projekt der {count} Projekt) | (Zeigt das aktivste Projekt der {count} Projekte)"
    },
    "submissionStates": "Einsendungenstatus",
    "other": "Anderes"
  },
  "es": {
    "title": "Informe de uso anonimizado",
    "introduction": [
      "Gracias por pensar en enviar informaci??n de uso. Estos datos nos ayudar??n a priorizar sus necesidades.",
      "Aqu?? se muestra el informe que estamos recopilando actualmente. Para responder a las nuevas funciones y necesidades, a veces cambiaremos lo que se informa, pero solo recopilaremos promedios resumidos como se ve aqu??.",
      "Siempre puedes venir aqu?? para ver qu?? se est?? recolectando."
    ],
    "projects": {
      "title": "Res??menes de proyectos",
      "subtitle": "(Mostrando el Proyecto m??s activo de {count} Proyecto) | (Mostrando el Proyecto m??s activo de {count} Proyectos) | (Mostrando el Proyecto m??s activo de {count} Proyectos)"
    },
    "submissionStates": "Estados de env??o",
    "other": "Otro"
  },
  "fr": {
    "title": "Rapport d'utilisation anonymis??",
    "introduction": [
      "Merci de penser ?? nous envoyer quelques informations d???utilisation. Ces donn??es nous aideront ?? prioriser vos besoins !",
      "Voici le rapport que nous recueillons actuellement. Pour r??pondre aux nouvelles fonctionnalit??s et aux nouveaux besoins, nous changerons parfois ce qui est rapport??, mais nous ne recueillerons jamais que des informations synth??tiques, comme celles que vous voyez ici.",
      "Vous pouvez toujours venir ici voir ce qui est collect??."
    ],
    "projects": {
      "title": "R??sum??s du projet",
      "subtitle": "(Affichage du projet le plus actif parmi {count} projets) | (Affichage du projet le plus actif parmi {count} projets) | (Affichage du projet le plus actif parmi {count} projets)"
    },
    "submissionStates": "??tats des soumissions",
    "datasets": {
      "title": "R??sum?? des Datasets et des Entit??s"
    },
    "other": "Autre"
  },
  "it": {
    "title": "Report di utilizzo anonimo",
    "introduction": [
      "Grazie per aver pensato di inviare alcune informazioni sull'utilizzo. Questi dati ci aiuteranno a dare la priorit?? alle tue esigenze!",
      "Qui ?? mostrato il rapporto che stiamo raccogliendo attualmente. Per rispondere a nuove funzionalit?? ed esigenze, a volte modificheremo ci?? che viene segnalato, ma raccoglieremo solo medie riepilogative come vedi qui.",
      "Puoi sempre venire qui per vedere cosa viene raccolto."
    ],
    "projects": {
      "title": "Riepiloghi del progetto",
      "subtitle": "(Visualizzazione del progetto pi?? attivo di {count} Progetto) | (Visualizzazione del progetto pi?? attivo di {count} Progetti) | (Visualizzazione del progetto pi?? attivo di {count} Progetti)"
    },
    "submissionStates": "Stato invio",
    "other": "Altro"
  },
  "ja": {
    "title": "????????????????????????????????????",
    "introduction": [
      "???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
      "???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
      "??????????????????????????????????????????????????????????????????????????????????????????"
    ],
    "projects": {
      "title": "???????????????????????????",
      "subtitle": "???????????????????????????{count}???????????????????????????????????????"
    },
    "submissionStates": "??????????????????????????????"
  },
  "sw": {
    "title": "Ripoti ya Matumizi Isiyojulikana",
    "introduction": [
      "Asante kwa kufikiria kutuma habari ya matumizi. Data hii itatusaidia kutanguliza mahitaji yako!",
      "Inayoonyeshwa hapa ni ripoti tunayokusanya kwa sasa. Ili kukabiliana na vipengele na mahitaji mapya, wakati mwingine tutabadilisha kile kinachoripotiwa, lakini tutawahi tu kukusanya wastani wa muhtasari kama unavyoona hapa.",
      "Unaweza kuja hapa kila wakati ili kuona kile kinachokusanywa."
    ],
    "projects": {
      "title": "Muhtasari wa Mradi",
      "subtitle": "(Inaonyesha Mradi amilifu zaidi wa Mradi ya {count}) | (Inaonyesha Mradi amilifu zaidi wa Miradi ya {count})"
    },
    "submissionStates": "Nchi za Uwasilishaji"
  }
}
</i18n>

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
  <div id="form-list">
    <page-section>
      <template #heading>
        <span>{{ $t('title') }}</span>
        <button v-if="project.dataExists && project.permits('form.create')"
          id="form-list-create-button" type="button" class="btn btn-primary"
          @click="showModal('newForm')">
          <span class="icon-plus-circle"></span>{{ $t('action.create') }}&hellip;
        </button>
        <form-sort v-model="sortMode"/>
      </template>
      <template #body>
        <form-table :sort-func="sortFunction"/>
        <form-table :sort-func="sortFunction" :show-closed="true"/>
        <loading :state="forms.initiallyLoading"/>
        <p v-if="forms.dataExists && forms.length === 0"
          class="empty-table-message">
          {{ $t('emptyTable') }}
        </p>
      </template>
    </page-section>
    <form-new v-bind="newForm" @hide="hideModal('newForm')"
      @success="afterCreate"/>
  </div>
</template>

<script>
import FormNew from './new.vue';
import FormTable from './table.vue';
import Loading from '../loading.vue';
import PageSection from '../page/section.vue';
import FormSort from './sort.vue';

import modal from '../../mixins/modal';
import routes from '../../mixins/routes';
import sortFunctions from '../../util/sort';
import { useRequestData } from '../../request-data';

export default {
  name: 'FormList',
  components: { FormTable, FormNew, FormSort, Loading, PageSection },
  mixins: [modal(), routes()],
  inject: ['alert'],
  setup() {
    // The component does not assume that this data will exist when the
    // component is created.
    const { project, forms } = useRequestData();
    return { project, forms };
  },
  data() {
    return {
      newForm: {
        state: false
      },
      sortMode: 'alphabetical'
    };
  },
  computed: {
    sortFunction() {
      return sortFunctions[this.sortMode];
    }
  },
  methods: {
    afterCreate(form) {
      const message = this.$t('alert.create', {
        name: form.name != null ? form.name : form.xmlFormId
      });
      this.$router.push(this.formPath(form.projectId, form.xmlFormId, 'draft'))
        .then(() => { this.alert.success(message); });
    }
  }
};
</script>

<style lang="scss">
#form-list { margin-bottom: 35px; }
</style>

<i18n lang="json5">
{
  "en": {
    // This is a title shown above a section of the page.
    "title": "Forms",
    "action": {
      // This is the text of a button that is used to create a new Form.
      "create": "New"
    },
    "emptyTable": "There are no Forms to show.",
    "alert": {
      "create": "Your new Form ???{name}??? has been created as a Draft. Take a look at the checklist below, and when you feel it???s ready, you can publish the Form for use."
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "Formul????e",
    "action": {
      "create": "Nov??"
    },
    "emptyTable": "Nejsou k dispozici ????dn?? formul????e.",
    "alert": {
      "create": "V???? nov?? formul???? ???{name}??? byl vytvo??en jako koncept. Pod??vejte se na od??krt??vac?? seznam n????e a a?? budete m??t pocit, ??e je p??ipraven, m????ete formul???? publikovat k pou??it??."
    }
  },
  "de": {
    "title": "Formulare",
    "action": {
      "create": "Neu"
    },
    "emptyTable": "Keine Formulare zum Anzeigen vorhanden.",
    "alert": {
      "create": "Ihr neues Formular \"{name}\" wurde als Entwurf erstellt. ??berpr??fen Sie die Checkliste unten: Wenn Sie finden, das sie abgearbeitet ist, k??nnen sie das Formular ver??ffentlichen."
    }
  },
  "es": {
    "title": "Formularios",
    "action": {
      "create": "Nuevo"
    },
    "emptyTable": "No hay formularios para mostrar.",
    "alert": {
      "create": "Su nuevo formulario \"{name}\" se ha creado como un borrador. De un vistazo a la lista de verificaci??n a continuaci??n, y cuando sienta que est?? lista, puede publicar el formulario para su uso."
    }
  },
  "fr": {
    "title": "Formulaires",
    "action": {
      "create": "Nouveau"
    },
    "emptyTable": "Il n'y a pas de formulaire ?? montrer.",
    "alert": {
      "create": "Votre nouveau formulaire \"{name}\" a ??t?? cr???? en tant qu'??bauche. Jetez un ??il ?? la liste de contr??le ci-dessous, et quand vous pensez que c'est pr??t, vous pouvez publier le formulaire pour utilisation."
    }
  },
  "id": {
    "title": "Formulir",
    "action": {
      "create": "Formulir Baru"
    },
    "emptyTable": "Tidak ada formulir untuk ditampilkan.",
    "alert": {
      "create": "Formulir {name} telah berhasil disimpan sebagai draf. Lihatlah daftar periksa di bawah, dan ketika Anda merasa sudah siap, Anda dapat menerbitkan formulir untuk digunakan."
    }
  },
  "it": {
    "title": "Formulari",
    "action": {
      "create": "Nuovo"
    },
    "emptyTable": "Non ci sono formulari da mostrare.",
    "alert": {
      "create": "Il tuo nuovo formulario\"{name}\" ?? stato creato come bozza. Dai un'occhiata alla lista di controllo qui sotto e quando ritieni che sia pronto, puoi pubblicare il formulario per l'uso."
    }
  },
  "ja": {
    "title": "????????????",
    "action": {
      "create": "????????????"
    },
    "emptyTable": "????????????????????????????????????????????????",
    "alert": {
      "create": "??????????????????\"{name}\"?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    }
  },
  "sw": {
    "title": "Fomu",
    "action": {
      "create": "Mpya"
    },
    "emptyTable": "hakuna Fomu za kuonyesha",
    "alert": {
      "create": "Fomu yako mpya \"{name}\" imeundwa kama Rasimu. Angalia orodha hapa chini, na unapohisi iko tayari, unaweza kuchapisha Fomu kwa matumizi."
    }
  }
}
</i18n>

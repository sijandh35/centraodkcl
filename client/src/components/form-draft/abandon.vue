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
  <modal id="form-draft-abandon" :state="state" :hideable="!awaitingResponse"
    backdrop @hide="$emit('hide')">
    <template #title>{{ title }}</template>
    <template #body>
      <div class="modal-introduction">
        <template v-if="form.publishedAt != null">
          <p>{{ $t('introduction.abandon[0]') }}</p>
          <p>{{ $t('introduction.abandon[1]') }}</p>
        </template>
        <template v-else>
          <p>{{ $t('introduction.deleteForm[0]') }}</p>
        </template>
        <p>{{ $t('common.areYouSure') }}</p>
      </div>
      <div class="modal-actions">
        <button type="button" class="btn btn-danger"
          :disabled="awaitingResponse" @click="abandon">
          {{ $t('action.abandon') }} <spinner :state="awaitingResponse"/>
        </button>
        <button type="button" class="btn btn-link" :disabled="awaitingResponse"
          @click="$emit('hide')">
          {{ $t('action.cancel') }}
        </button>
      </div>
    </template>
  </modal>
</template>

<script>
import Modal from '../modal.vue';
import Spinner from '../spinner.vue';

import request from '../../mixins/request';
import { apiPaths } from '../../util/request';
import { noop } from '../../util/util';
import { useRequestData } from '../../request-data';

export default {
  name: 'FormDraftAbandon',
  components: { Modal, Spinner },
  mixins: [request()],
  props: {
    state: {
      type: Boolean,
      default: false
    }
  },
  emits: ['hide', 'success'],
  setup() {
    // The component assumes that this data will exist when the component is
    // created.
    const { form } = useRequestData();
    return { form };
  },
  data() {
    return {
      awaitingResponse: false
    };
  },
  computed: {
    title() {
      return this.form.publishedAt != null
        ? this.$t('title.abandon')
        : this.$t('title.deleteForm');
    }
  },
  methods: {
    abandon() {
      this.request({
        method: 'DELETE',
        url: this.form.publishedAt != null
          ? apiPaths.formDraft(this.form.projectId, this.form.xmlFormId)
          : apiPaths.form(this.form.projectId, this.form.xmlFormId)
      })
        .then(() => {
          // project.forms and project.lastSubmission may now be out-of-date. If
          // the user navigates to ProjectOverview, project.forms should be
          // updated. project.lastSubmission is not used within ProjectShow.
          this.$emit('success');
        })
        .catch(noop);
    }
  }
};
</script>

<i18n lang="json5">
{
  "en": {
    // This is the title at the top of a pop-up.
    "title": {
      "abandon": "Abandon Draft",
      "deleteForm": "Abandon Draft and Delete Form"
    },
    "introduction": {
      "abandon": [
        "You are about to permanently delete the Draft version of this Form. This means that the draft Form definition, any draft Form Attachments you have uploaded, and all test Submissions will be removed.",
        "Your published Form definition, its Form Attachments, and Submissions will not be affected."
      ],
      "deleteForm": [
        "You are about to delete this draft Form definition, along with any draft Form Attachments you have uploaded, and all test Submissions. Because you have not yet published it, this entire Form will be deleted and moved to the Trash."
      ]
    },
    "action": {
      "abandon": "Abandon"
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": {
      "abandon": "Opustit koncept",
      "deleteForm": "Opustit koncept a smazat formul????"
    },
    "introduction": {
      "abandon": [
        "Chyst??te se trvale odstranit pracovn?? verzi tohoto formul????e. To znamen??, ??e definice n??vrhu formul????e, v??echny v??mi nahran?? p????lohy n??vrhu formul????e a v??echna testovac?? pod??n?? budou odstran??na.",
        "Va??e zve??ejn??n?? definice formul????e, jeho p????lohy a pod??n?? nebudou ovlivn??ny."
      ],
      "deleteForm": [
        "Chyst??te se odstranit tento n??vrh definice formul????e spolu se v??emi nahran??mi n??vrhy p????loh formul????e a v??emi testovac??mi pod??n??mi. Proto??e jste jej je??t?? nezve??ejnili, bude cel?? tento Formul???? odstran??n a p??esunut do ko??e."
      ]
    },
    "action": {
      "abandon": "Opustit"
    }
  },
  "de": {
    "title": {
      "abandon": "Entwurf verwerfen",
      "deleteForm": "Entwurf verwerfen und Formular l??schen"
    },
    "action": {
      "abandon": "Ja, L??schen"
    }
  },
  "es": {
    "title": {
      "abandon": "Abandonar borrador",
      "deleteForm": "Abandonar borrador y borrar formulario"
    },
    "action": {
      "abandon": "Abandonar"
    }
  },
  "fr": {
    "title": {
      "abandon": "Abandonner l'??bauche",
      "deleteForm": "Abandonner l'??bauche et supprimer le formulaire"
    },
    "introduction": {
      "abandon": [
        "Vous ??tes sur le point de supprimer d??finitivement l'??bauche de ce formulaire. Cela signifie que sa d??finition, les pi??ces jointes associ??es que vous avez t??l??charg??es et toutes les soumissions de test seront supprim??es.",
        "Votre d??finition de Formulaire publi??e, ses Fichiers joints, et les Soumissions ne seront pas affect??s."
      ],
      "deleteForm": [
        "Vous ??tes sur le point de supprimer la d??finition de cette ??bauche de Formulaire, ainsi que chaque Fichier joint envoy??s, et toutes les Soumissions de test. Parce-que vous ne l'avez pas encore publi??, tout le formulaire sera supprim?? et d??plac?? ?? la corbeille."
      ]
    },
    "action": {
      "abandon": "Abandonner"
    }
  },
  "id": {
    "title": {
      "abandon": "Buang Draf",
      "deleteForm": "Buang Draf dan Hapus Formulir"
    },
    "action": {
      "abandon": "Buang"
    }
  },
  "it": {
    "title": {
      "abandon": "Abbandona bozza",
      "deleteForm": "Abbandona ed elimina la bozza"
    },
    "action": {
      "abandon": "Annulla"
    }
  },
  "ja": {
    "title": {
      "abandon": "??????????????????",
      "deleteForm": "?????????????????????????????????"
    },
    "action": {
      "abandon": "??????"
    }
  },
  "sw": {
    "title": {
      "abandon": "acha rasimu",
      "deleteForm": "Achana na Rasimu na ufute Fomu"
    },
    "action": {
      "abandon": "Achana"
    }
  }
}
</i18n>

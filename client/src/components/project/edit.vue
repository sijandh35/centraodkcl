<!--
Copyright 2019 ODK Central Developers
See the NOTICE file at the top-level directory of this distribution and at
https://github.com/getodk/central-frontend/blob/master/NOTICE.

This file is part of ODK Central. It is subject to the license terms in
the LICENSE file found in the top-level directory of this distribution and at
https://www.apache.org/licenses/LICENSE-2.0. No part of ODK Central,
including this file, may be copied, modified, propagated, or distributed
except according to the terms contained in the LICENSE file.
-->
<template>
  <div id="project-edit" class="panel panel-simple">
    <div class="panel-heading">
      <h1 class="panel-title">{{ $t('title') }}</h1>
    </div>
    <div class="panel-body">
      <form @submit.prevent="submit">
        <form-group v-model.trim="name" :placeholder="$t('field.name')" required
          autocomplete="off"/>
        <!-- eslint-disable-next-line vuejs-accessibility/label-has-for -->
        <label class="form-group">
          <markdown-textarea v-model="description" rows="10"/>
          <span class="form-label">{{ $t('field.description') }}</span>
          <span class="note">{{ $t('field.note') }}</span>
        </label>
        <button type="submit" class="btn btn-primary"
          :disabled="awaitingResponse">
          {{ $t('action.saveSettings') }} <spinner :state="awaitingResponse"/>
        </button>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ProjectEdit'
};
</script>
<script setup>
import { inject, ref } from 'vue';
import { useI18n } from 'vue-i18n';

import FormGroup from '../form-group.vue';
import Spinner from '../spinner.vue';
import MarkdownTextarea from '../markdown/textarea.vue';

import { apiPaths } from '../../util/request';
import { noop } from '../../util/util';
import { useRequestData } from '../../request-data';

// The component assumes that this data will exist when the component is
// created.
const { project } = useRequestData();
const { awaitingResponse } = project.toRefs();

const name = ref(project.name);
const description = ref(project.description);

const { t } = useI18n();
const alert = inject('alert');
const submit = () => {
  project.request({
    method: 'PATCH',
    url: apiPaths.project(project.id),
    data: { name: name.value, description: description.value },
    patch: ({ data }) => {
      project.name = data.name;
      project.description = data.description;
      project.updatedAt = data.updatedAt;
    }
  })
    .then(() => { alert.success(t('alert.success')); })
    .catch(noop);
};
</script>

<style lang="scss">
#project-edit {
  .note {
    float: right;
    color: #333;
    font-size: 11px;
    padding-right: 12px;
    position: relative;
    top: 2px;
  }
}
</style>

<i18n lang="json5">
{
  "en": {
    // This is a title shown above a section of the page.
    "title": "Basic Details",
    "field": {
      "name": "Project name",
      "description": "Project description",
      "note": "Add notes, links, instructions and other resources here."
    },
    "alert": {
      "success": "Project settings saved!"
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "Z??kladn?? detaily",
    "field": {
      "name": "N??zev projektu",
      "description": "Popis Projektu",
      "note": "Sem p??idejte pozn??mky, odkazy, n??vody a dal???? zdroje."
    },
    "alert": {
      "success": "Nastaven?? projektu ulo??eno."
    }
  },
  "de": {
    "title": "Basisinformationen",
    "field": {
      "name": "Projektname",
      "description": "Projektbeschreibung",
      "note": "F??gen Sie hier Notizen, Links, Anweisungen und andere Ressourcen hinzu."
    },
    "alert": {
      "success": "Projekteinstellungen wurden gespeichert"
    }
  },
  "es": {
    "title": "Detalles b??sicos",
    "field": {
      "name": "Nombre del proyecto",
      "description": "Descripci??n del Proyecto",
      "note": "Agregue notas, enlaces, instrucciones y otros recursos aqu??."
    },
    "alert": {
      "success": "??Configuraci??n del proyecto guardada!"
    }
  },
  "fr": {
    "title": "D??tails basiques",
    "field": {
      "name": "Nom du projet",
      "description": "Description du projet",
      "note": "Ajoutez des notes, des liens, des instructions ou autres ressources ici."
    },
    "alert": {
      "success": "Param??tres du projet enregistr??s !"
    }
  },
  "id": {
    "title": "Rincian Dasar",
    "field": {
      "name": "Nama Proyek"
    },
    "alert": {
      "success": "Pengaturan proyek tersimpan!"
    }
  },
  "it": {
    "title": "Dettagli di base",
    "field": {
      "name": "Nome del Progetto",
      "description": "Descrizione del progetto",
      "note": "Aggiungi note, collegamenti, istruzioni e altre risorse qui."
    },
    "alert": {
      "success": "Impostazioni del progetto salvate!"
    }
  },
  "ja": {
    "title": "????????????",
    "field": {
      "name": "?????????????????????"
    },
    "alert": {
      "success": "??????????????????????????????????????????"
    }
  },
  "sw": {
    "title": "Maelezo ya Msingi",
    "field": {
      "name": "Jina la mradi"
    },
    "alert": {
      "success": "Mipangilio ya mradi imehifadhiwa!"
    }
  }
}
</i18n>

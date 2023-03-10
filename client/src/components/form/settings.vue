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
  <div v-if="form.dataExists" id="form-settings">
    <div class="row">
      <div class="col-xs-6">
        <div class="panel panel-simple">
          <div class="panel-heading">
            <h1 class="panel-title">{{ $t('state.title') }}</h1>
          </div>
          <div class="panel-body">
            <i18n-t tag="p" keypath="state.body.full">
              <template #formAccessSettings>
                <router-link :to="projectPath('form-access')">{{ $t('state.body.formAccessSettings') }}</router-link>
              </template>
            </i18n-t>
          </div>
        </div>
      </div>
      <div class="col-xs-6">
        <div class="panel panel-simple-danger">
          <div class="panel-heading">
            <h1 class="panel-title">{{ $t('common.dangerZone') }}</h1>
          </div>
          <div class="panel-body">
            <p>
              <button type="button" class="btn btn-danger"
                @click="showModal('deleteForm')">
                {{ $t('action.delete') }}&hellip;
              </button>
            </p>
          </div>
        </div>
      </div>
    </div>
    <form-delete v-bind="deleteForm" @hide="hideModal('deleteForm')"
      @success="afterDelete"/>
  </div>
</template>

<script>
import FormDelete from './delete.vue';

import modal from '../../mixins/modal';
import routes from '../../mixins/routes';
import { useRequestData } from '../../request-data';

export default {
  name: 'FormSettings',
  components: { FormDelete },
  mixins: [modal(), routes()],
  inject: ['alert'],
  setup() {
    const { form } = useRequestData();
    return { form };
  },
  data() {
    return {
      deleteForm: {
        state: false
      }
    };
  },
  methods: {
    afterDelete() {
      const message = this.$t('alert.delete', { name: this.form.nameOrId });
      this.$router.push(this.projectPath())
        .then(() => { this.alert.success(message); });
    }
  }
};
</script>

<style lang="scss">
#form-settings .panel-simple-danger .panel-body p {
  margin-bottom: 15px;
  margin-top: 10px;
  text-align: center;
}
</style>

<i18n lang="json5">
{
  "en": {
    "state": {
      // This is a title shown above a section of the page.
      "title": "Form State",
      "body": {
        "full": "To set this Form???s state, please visit the Project {formAccessSettings}.",
        "formAccessSettings": "Form Access settings"
      }
    },
    "action": {
      "delete": "Delete this Form"
    },
    "alert": {
      "delete": "The Form ???{name}??? was deleted."
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "state": {
      "title": "Stav formul????e",
      "body": {
        "full": "Chcete-li nastavit stav tohoto formul????e, nav??tivte {formAccessSettings} Projektu.",
        "formAccessSettings": "Nastaven?? p????stupu k formul????i"
      }
    },
    "action": {
      "delete": "Odstranit tento formul????"
    },
    "alert": {
      "delete": "Formul???? ???{name}??? byl odstran??n."
    }
  },
  "de": {
    "state": {
      "title": "Formular-Status",
      "body": {
        "full": "Die Zugriffsberechtigungen f??r diese Formular finden Sie in {formAccessSettings}.",
        "formAccessSettings": "Zugriffsberechtigungen"
      }
    },
    "action": {
      "delete": "Dieses Formular l??schen"
    },
    "alert": {
      "delete": "Das Formular \"{name}\" wurde gel??scht."
    }
  },
  "es": {
    "state": {
      "title": "Estado del formulario",
      "body": {
        "full": "Para establecer el estado de este formulario, por favor visite la {formAccessSettings} de este proyecto.",
        "formAccessSettings": "configuraci??n de acceso a formulario"
      }
    },
    "action": {
      "delete": "Borrar este formulario"
    },
    "alert": {
      "delete": "El formulario \"{name}\" fue borrado."
    }
  },
  "fr": {
    "state": {
      "title": "??tat du formulaire",
      "body": {
        "full": "Pour d??finir l'??tat de ce formulaire, veuillez consulter le {formAccessSettings} du projet.",
        "formAccessSettings": "Param??tres d'acc??s aux formualires"
      }
    },
    "action": {
      "delete": "Supprimer ce formulaire"
    },
    "alert": {
      "delete": "Le formulaire {name} a ??t?? supprim??."
    }
  },
  "id": {
    "state": {
      "title": "Status Formulir",
      "body": {
        "full": "Untuk mengatur status formulir, silakan kunjungi {formAccessSettings} Proyek.",
        "formAccessSettings": "Pengaturan Akses Formulir"
      }
    },
    "action": {
      "delete": "Hapus formulir ini"
    },
    "alert": {
      "delete": "Formulir {name} telah dihapus."
    }
  },
  "it": {
    "state": {
      "title": "Stato del Formulario",
      "body": {
        "full": "Per impostare lo stato di questo formulario, visita il progetto {formAccessSettings}",
        "formAccessSettings": "Configurazioni di accesso al form"
      }
    },
    "action": {
      "delete": "Cancella questo formulario"
    },
    "alert": {
      "delete": "Il Formulario ???{name}??? ?? stato cancellato."
    }
  },
  "ja": {
    "state": {
      "title": "?????????????????????",
      "body": {
        "full": "???????????????????????????????????????????????????????????????????????????{formAccessSettings}???????????????????????????",
        "formAccessSettings": "?????????????????????????????????"
      }
    },
    "action": {
      "delete": "?????????????????????"
    },
    "alert": {
      "delete": "????????????\"{name}\"???????????????????????????"
    }
  },
  "sw": {
    "state": {
      "title": "Jimbo la Fomu",
      "body": {
        "full": "Ili kuweka hali ya Fomu hii, tafadhali tembelea Project {formAccessSettings}.",
        "formAccessSettings": "Mipangilio ya Ufikiaji wa Fomu"
      }
    },
    "action": {
      "delete": "Futa Fomu hii"
    },
    "alert": {
      "delete": "Fomu \"{name}\" ilifutwa."
    }
  }
}
</i18n>

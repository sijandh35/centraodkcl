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

<!-- Although Backend supports more complex use cases, we assume in this
component that each user is assigned only one role and that further, each user
either is an Administrator or has no role. -->
<template>
  <div>
    <div class="heading-with-button">
      <button id="user-list-new-button" type="button" class="btn btn-primary"
        @click="showModal('newUser')">
        <span class="icon-plus-circle"></span>{{ $t('action.create') }}&hellip;
      </button>
      <i18n-t tag="p" keypath="heading[0]">
        <template #collect>
          <doc-link to="collect-intro/">ODK Collect</doc-link>
        </template>
      </i18n-t>
    </div>
    <table id="user-list-table" class="table">
      <thead>
        <tr>
          <th>{{ $t('header.displayName') }}</th>
          <th>{{ $t('header.email') }}</th>
          <th>{{ $t('header.sitewideRole') }}</th>
          <th class="actions">{{ $t('header.actions') }}</th>
        </tr>
      </thead>
      <tbody v-if="dataExists">
        <user-row v-for="user of users" :key="user.id" :user="user"
          :admin="adminIds.has(user.id)" :highlighted="highlighted"
          @assigned-role="afterAssignRole" @reset-password="showResetPassword"
          @retire="showRetire"/>
      </tbody>
    </table>
    <loading :state="initiallyLoading"/>

    <user-new v-bind="newUser" @hide="hideModal('newUser')"
      @success="afterCreate"/>
    <user-reset-password v-bind="resetPassword" @hide="hideResetPassword"
      @success="afterResetPassword"/>
    <user-retire v-bind="retire" @hide="hideRetire" @success="afterRetire"/>
  </div>
</template>

<script>
import DocLink from '../doc-link.vue';
import Loading from '../loading.vue';
import UserNew from './new.vue';
import UserResetPassword from './reset-password.vue';
import UserRetire from './retire.vue';
import UserRow from './row.vue';

import modal from '../../mixins/modal';
import { useRequestData } from '../../request-data';

export default {
  name: 'UserList',
  components: {
    DocLink,
    Loading,
    UserNew,
    UserResetPassword,
    UserRetire,
    UserRow
  },
  mixins: [modal()],
  inject: ['alert'],
  setup() {
    const { createResource, resourceStates } = useRequestData();
    const users = createResource('users');
    const adminIds = createResource('adminIds', () => ({
      transformResponse: ({ data }) =>
        data.reduce((set, actor) => set.add(actor.id), new Set())
    }));
    return { users, adminIds, ...resourceStates([users, adminIds]) };
  },
  data() {
    return {
      // The id of the highlighted user
      highlighted: null,
      // Modals
      newUser: {
        state: false
      },
      resetPassword: {
        state: false,
        user: null
      },
      retire: {
        state: false,
        user: null
      }
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      Promise.allSettled([
        this.users.request({ url: '/v1/users' }),
        this.adminIds.request({ url: '/v1/assignments/admin' })
      ]);
    },
    afterCreate(user) {
      this.fetchData();
      this.hideModal('newUser');
      this.alert.success(this.$t('alert.create', user));
      this.highlighted = user.id;
    },
    // Called after a user is assigned a new role (including None).
    afterAssignRole(user, admin) {
      this.alert.success(this.$t('alert.assignRole', {
        displayName: user.displayName,
        roleName: admin ? this.$t('role.admin') : this.$t('role.none')
      }));

      /*
      Here we update this.adminIds. If the current user has also refreshed the
      data, then in rare cases, the update to this.adminIds will not be shown.
      For example:

        1. The current user changes another user's sitewide role.
        2. Immediately after, the user clicks the refresh button.
        3. The response for the role change is received first, updating
           this.adminIds.
        4. The response for the refresh is received, possibly undoing the
           update to this.adminIds.

      In cases like this, the current user will likely refresh Frontend or redo
      the change.
      */
      if (this.adminIds.dataExists) {
        if (admin) {
          this.adminIds.add(user.id);
        } else {
          this.adminIds.delete(user.id);
        }
      }
    },
    showResetPassword(user) {
      this.resetPassword.user = user;
      this.showModal('resetPassword');
    },
    hideResetPassword() {
      this.hideModal('resetPassword');
      this.resetPassword.user = null;
    },
    afterResetPassword(user) {
      this.hideResetPassword();
      this.hideModal('resetPassword');
      this.alert.success(this.$t('alert.resetPassword', user));
    },
    showRetire(user) {
      this.retire.user = user;
      this.showModal('retire');
    },
    hideRetire() {
      this.hideModal('retire');
      this.retire.user = null;
    },
    afterRetire(user) {
      this.fetchData();
      this.hideRetire();
      this.alert.success(this.$t('alert.retire', user));
      this.highlighted = null;
    }
  }
};
</script>

<style lang="scss">
#user-list-table {
  table-layout: fixed;

  th.actions { width: 125px; }
}
</style>

<i18n lang="json5">
{
  "en": {
    "action": {
      "create": "Create Web User"
    },
    "heading": [
      // {collect} is a link whose text is "ODK Collect".
      "Web Users have accounts on this website to oversee and administer the Projects on this server. Administrators can manage anything on the site. Users with no Sitewide Role can be given a Role on any Project, from that Project???s settings. Sitewide Administrators and some Project Roles can use a web browser to fill out Forms. To submit data through an application such as {collect}, create App Users for each Project."
    ],
    "header": {
      "sitewideRole": "Sitewide Role"
    },
    "alert": {
      "create": "A user was created successfully for ???{displayName}???.",
      "assignRole": "Success! ???{displayName}??? has been given a Sitewide Role of ???{roleName}???.",
      "resetPassword": "The password for ???{displayName}??? has been invalidated. An email has been sent to {email} with instructions on how to proceed.",
      "retire": "The user ???{displayName}??? has been retired."
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "action": {
      "create": "Vytvo??it webov??ho u??ivatele"
    },
    "heading": [
      "Webov?? u??ivatel?? maj?? na t??to webov?? str??nce ????ty pro p??ehled a spr??vu projekt?? na tomto serveru. Spr??vci mohou na webu spravovat cokoli. U??ivatel?? bez role ??pln??ho p????stupu mohou m??t roli v jak??mkoli projektu, z jeho Projektov??ho nastaven??. Administr??to??i webu a n??kter?? role projektu mohou k vypln??n?? formul?????? pou????t webov?? prohl????e??. Chcete-li odeslat data prost??ednictv??m aplikace, jako {collect} vytvo??te u??ivatele aplikace pro ka??d?? projekt."
    ],
    "header": {
      "sitewideRole": "Role ??pln??ho p????stupu"
    },
    "alert": {
      "create": "U??ivatel byl ??sp????n?? vytvo??en pro ???{displayName}???.",
      "assignRole": "??sp??ch! ???{displayName}??? z??skal roli ??pln??ho p????stupu ???{roleName}???.",
      "resetPassword": "Heslo pro ???{displayName}??? bylo zneplatn??no. Byl odesl??n e-mail na {email} s pokyny, jak postupovat.",
      "retire": "U??ivatel ???{displayName}??? byl usp??n."
    }
  },
  "de": {
    "action": {
      "create": "Web-Benutzer anlegen"
    },
    "heading": [
      "Web-Benutzer haben Konten auf dieser Webseite, um Projekte auf diesem Server zu ??berwachen und zu administrieren. Administratoren k??nnen alles auf dieser Seite verwalten. Benutzer ohne seitenweite Rolle k??nnen in jedem Projekt in den Projekteinstellungen eine Rolle zugewiesen bekommen. Seitenweite Administratoren und einige Projektrollen k??nnen einen Webbrowser zum Ausf??llen von Formularen benutzen. Um Daten ??ber eine App wie {collect} zu ??bermitteln, erstellen Sie App-Benutzer f??r jedes Projekt."
    ],
    "header": {
      "sitewideRole": "Seitenweite Rolle"
    },
    "alert": {
      "create": "Ein Benutzer f??r \"{displayName}\" wurde erfolgreich erstellt.",
      "assignRole": "\"{displayName}\" wurde erfolgreich eine seitenweite Rolle als \"{roleName}\" zugewiesen.",
      "resetPassword": "Das Passwort f??r \"{displayName}\" wurde als ung??ltig gekennzeichnet. Eine E-Mail wurde an {email} gesendet mit Anweisungen wie weiter vorgegangen wird.",
      "retire": "Der App-Benutzer \"{displayName}\" wurde deaktiviert."
    }
  },
  "es": {
    "action": {
      "create": "Crear usuario web"
    },
    "heading": [
      "Los usuarios web tienen cuentas en este sitio web, para supervisar y administrar proyectos en el servidor. Los administradores pueden gestionar cualquier cosa en la p??gina. Los usuarios que no poseen un Rol, se le puede asignar un Rol en configuraci??n de proyecto. Los Administradores y algunos roles en el proyecto, pueden usar un explorador web para completar formularios. Para enviar la informaci??n a trav??s de una aplicaci??n tal como {collect}, cree Usuarios m??viles en cada proyecto."
    ],
    "header": {
      "sitewideRole": "Rol de sitio"
    },
    "alert": {
      "create": "Se cre?? un usuario satisfactoriamente para ???{displayName}???.",
      "assignRole": "Realizado. Al usuario \"{displayName}\" se le ha asignado el rol \" {roleName}\" en este proyecto.",
      "resetPassword": "La contrase??a asociada al usuario {displayName} es incorrecta. Se ha enviado un correo electr??nico a {email} con las instrucciones para continuar.",
      "retire": "El usuario \"{displayName}\" ha sido retirado"
    }
  },
  "fr": {
    "action": {
      "create": "Cr??er un utilisateur web"
    },
    "heading": [
      "Les utilisateurs web ont des comptes sur ce site pour superviser et administrer les projets sur ce serveur. Les administrateurs peuvent g??rer tout ce qui se trouve sur le site. Les utilisateurs n'ayant aucun r??le sur l'ensemble du site peuvent toujours ??tre nomm??s gestionnaires de projet pour n'importe quel projet, ?? partir des param??tres de ce projet. Les administrateurs sur l'ensemble du site et certains r??les de projet peuvent utiliser leurs navigateurs pour remplir des formulaires. Pour soumettre des donn??es ?? travers une application telle que {collect}, cr??ez des utilisateurs mobiles pour chaque projet."
    ],
    "header": {
      "sitewideRole": "R??le sur l'ensemble du site"
    },
    "alert": {
      "create": "Un utilisateur a ??t?? cr???? avec succ??s pour \"{displayName}\".",
      "assignRole": "Succ??s ! \"{displayName}\" a re??u le r??le de \"{roleName}\" pour l'ensemble du site.",
      "resetPassword": "Le mot de passe pour \"{displayName}\" a ??t?? invalid??. Un courrier ??lectronique a ??t?? envoy?? ?? {email} avec des instructions sur la mani??re de proc??der.",
      "retire": "L'utilisateur {displayName} a ??t?? supprim??."
    }
  },
  "id": {
    "action": {
      "create": "Buat Pengguna Web"
    },
    "heading": [
      "Pengguna Web memiliki akun di website ini untuk mengawasi dan mengelola Proyek pada server ini. Administrator dapat mengelola apapun di situs. Pengguna yang tidak memiliki Peran Seluruh Situs bisa diberikan Peran dalam Proyek apapun, melalui pengaturan Proyek tersebut. Administrator Seluruh Sirus dan beberapa Perak Proyek dapat menggunakan web browser untuk mengisi Formulir. Untuk mengirim data lewat aplikasi seperti {collect}, buat Pengguna Aplikasi untuk masing-masing Proyek."
    ],
    "header": {
      "sitewideRole": "Peran Seluruh Situs"
    },
    "alert": {
      "create": "Pengguna telah sukses dibuat untuk \"{displayName}\".",
      "assignRole": "Berhasil! \"{displayName}\" telah diberikan Peran Seluruh Situs oleh \"{roleName}\".",
      "resetPassword": "Kata sandi untuk \"{displayName}\" sudah tidak berlaku. Sebuah email tetang instruksi selanjutnya telah dikirimkan ke {email}.",
      "retire": "Pengguna {displayName} telah berhenti."
    }
  },
  "it": {
    "action": {
      "create": "Crea un Utente Web"
    },
    "heading": [
      "Gli Utenti Web hanno degli account su questo sito per supervisionare e amministrare i Progetti sul server. Gli amministratori possono gestire qualsiasi cosa sul sito. Agli utenti senza ruolo di supervisione e amministrazione pu?? essere assegnato un ruolo su qualsiasi progetto, attraverso le impostazioni del progetto stesso. Gli amministratori del sito e alcuni ruoli del progetto possono utilizzare un browser Web per compilare i formulari. Per inviare dati tramite un'applicazione come {collect}, create degli Utenti app per ciascun progetto."
    ],
    "header": {
      "sitewideRole": "Ruolo per tutto il sito"
    },
    "alert": {
      "create": "Un utente ?? stato creato con successo per \"{displayName}\"",
      "assignRole": "Successo! A \"{displayName}\" ?? stato assegnato un ruolo di \"{roleName}\" in tutto il sito.",
      "resetPassword": "La password per \"{displayName}\" ?? stata invalidata. ?? stata inviata un'e-mail a {email} con le istruzioni su come procedere.",
      "retire": "L'utente \"{displayName}\" ?? stato ritirato."
    }
  },
  "ja": {
    "action": {
      "create": "Web?????????????????????"
    },
    "heading": [
      "Web???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????Web???????????????????????????????????????????????????????????????{collect}????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    ],
    "header": {
      "sitewideRole": "????????????????????????"
    },
    "alert": {
      "create": "????????????\"{displayName}\"?????????????????????????????????",
      "assignRole": "???????????????\"{displayName}\"???????????????????????????\"{roleName}\"???????????????????????????",
      "resetPassword": "\"{displayName}\"????????????????????????????????????{email}?????????????????????????????????????????????????????????????????????????????????????????????????????????",
      "retire": "????????????\"{displayName}\"????????????????????????"
    }
  },
  "sw": {
    "action": {
      "create": "Unda Mtumiaji wa Wavuti"
    },
    "heading": [
      "Watumiaji Wavuti wana akaunti kwenye tovuti hii ili kusimamia na kusimamia Miradi kwenye seva hii. Wasimamizi wanaweza kudhibiti chochote kwenye tovuti. Watumiaji wasio na Jukumu la Tovuti Pote wanaweza kupewa Jukumu kwenye Mradi wowote, kutoka kwa mipangilio ya Mradi huo. Wasimamizi wa Tovuti nzima na baadhi ya Majukumu ya Mradi wanaweza kutumia kivinjari cha wavuti kujaza Fomu. Ili kuwasilisha data kupitia programu kama vile {collect}, unda Watumiaji wa Programu kwa kila Mradi."
    ],
    "header": {
      "sitewideRole": "Jukumu la Tovuti nzima"
    },
    "alert": {
      "create": "Mtumiaji ameundwa kwa ajili ya \"{displayName}\".",
      "assignRole": "Mafanikio! \"{displayName}\" imepewa Jukumu la Eneo Lote la \"{roleName}\".",
      "resetPassword": "Nenosiri la \"{displayName}\" limebatilishwa. Barua pepe imetumwa kwa {email} ikiwa na maagizo ya jinsi ya kuendelea.",
      "retire": "Mtumiaji \"{displayName}\" amestaafu."
    }
  }
}
</i18n>

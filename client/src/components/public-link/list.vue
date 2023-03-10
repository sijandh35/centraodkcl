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
  <div id="public-link-list">
    <div class="heading-with-button">
      <button type="button" class="btn btn-primary"
        @click="showModal('create')">
        <span class="icon-plus-circle"></span>{{ $t('action.create') }}&hellip;
      </button>
      <p>
        <i18n-t keypath="heading[0].full">
          <template #state>
            <router-link :to="projectPath('form-access')">{{ $t('heading[0].state') }}</router-link>
          </template>
        </i18n-t>
        <sentence-separator/>
        <i18n-t keypath="moreInfo.clickHere.full">
          <template #clickHere>
            <doc-link to="central-submissions/#public-access-links">{{ $t('moreInfo.clickHere.clickHere') }}</doc-link>
          </template>
        </i18n-t>
      </p>
      <i18n-t tag="p" keypath="heading[1].full">
        <template #clickHere>
          <a href="#" @click.prevent="showModal('submissionOptions')">{{ $t('heading[1].clickHere') }}</a>
        </template>
      </i18n-t>
    </div>

    <public-link-table :highlighted="highlighted" @revoke="showRevoke"/>
    <loading :state="publicLinks.initiallyLoading"/>
    <p v-if="publicLinks.dataExists && publicLinks.length === 0"
      class="empty-table-message">
      {{ $t('emptyTable') }}
    </p>

    <public-link-create v-bind="create" @hide="hideModal('create')"
      @success="afterCreate"/>
    <project-submission-options v-bind="submissionOptions"
      @hide="hideModal('submissionOptions')"/>
    <public-link-revoke v-bind="revoke" @hide="hideRevoke"
      @success="afterRevoke"/>
  </div>
</template>

<script>
import DocLink from '../doc-link.vue';
import Loading from '../loading.vue';
import ProjectSubmissionOptions from '../project/submission-options.vue';
import PublicLinkCreate from './create.vue';
import PublicLinkRevoke from './revoke.vue';
import PublicLinkTable from './table.vue';
import SentenceSeparator from '../sentence-separator.vue';

import modal from '../../mixins/modal';
import routes from '../../mixins/routes';
import { apiPaths } from '../../util/request';
import { noop } from '../../util/util';
import { useRequestData } from '../../request-data';

export default {
  name: 'PublicLinkList',
  components: {
    DocLink,
    Loading,
    ProjectSubmissionOptions,
    PublicLinkCreate,
    PublicLinkRevoke,
    PublicLinkTable,
    SentenceSeparator
  },
  mixins: [modal(), routes()],
  inject: ['alert'],
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
    const { publicLinks } = useRequestData();
    return { publicLinks };
  },
  data() {
    return {
      // The id of the highlighted public link
      highlighted: null,
      // Modals
      create: {
        state: false
      },
      submissionOptions: {
        state: false
      },
      revoke: {
        state: false,
        publicLink: null
      }
    };
  },
  created() {
    this.fetchData(false);
  },
  methods: {
    fetchData(resend) {
      this.publicLinks.request({
        url: apiPaths.publicLinks(this.projectId, this.xmlFormId),
        resend
      }).catch(noop);
      this.highlighted = null;
    },
    showRevoke(publicLink) {
      this.revoke.publicLink = publicLink;
      this.showModal('revoke');
    },
    hideRevoke() {
      this.hideModal('revoke');
      this.revoke.publicLink = null;
    },
    afterCreate(publicLink) {
      this.fetchData(true);
      this.hideModal('create');
      this.alert.success(this.$t('alert.create'));
      this.highlighted = publicLink.id;
    },
    afterRevoke(publicLink) {
      this.fetchData(true);
      this.hideRevoke();
      this.alert.success(this.$t('alert.revoke', publicLink));
    }
  }
};
</script>

<i18n lang="json5">
{
  "en": {
    "action": {
      "create": "Create Public Access Link",
    },
    "heading": [
      {
        "full": "Anyone with a Public Access Link can fill out this Form in a web browser. You can create multiple Links to track different distributions of the Form, to limit how long a specific group of people has access to the Form, and more. These links will only work if the Form is in the Open {state}.",
        "state": "state"
      },
      {
        "full": "Public Links are intended for self-reporting. If you are working with data collectors who need to submit the same Form multiple times, {clickHere} for other options.",
        "clickHere": "click here"
      }
    ],
    "emptyTable": "There are no Public Access Links for this Form.",
    "alert": {
      "create": "Success! Your Public Access Link has been created and is now live. Copy it below to distribute it.",
      "revoke": "The Public Access Link ???{displayName}??? was revoked successfully. No further Submissions will be accepted using this Link."
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "action": {
      "create": "Vytvo??it ve??ejn?? p????stupn?? odkaz"
    },
    "heading": [
      {
        "full": "Kdokoli s odkazem na ve??ejn?? p????stup m????e vyplnit tento formul???? ve webov??m prohl????e??i. M????ete vytvo??it v??ce odkaz?? pro sledov??n?? r??zn??ch distribuc?? formul????e, abyste omezili, jak dlouho m?? ur??it?? skupina lid?? p????stup k formul????i a dal????. Tyto odkazy budou fungovat, pouze pokud je formul???? ve {state} Open.",
        "state": "stavu"
      },
      {
        "full": "Ve??ejn?? odkazy jsou ur??eny pro vlastn?? reportov??n??. Pokud pracujete se sb??ra??i dat, kte???? pot??ebuj?? zaslat stejn?? formul???? v??cekr??t, {clickHere} pro jin?? mo??nosti.",
        "clickHere": "klikn??te zde"
      }
    ],
    "emptyTable": "Pro tento formul???? neexistuj?? ????dn?? ve??ejn?? p????stupn?? odkazy.",
    "alert": {
      "create": "??sp??ch! V???? ve??ejn?? p????stupn?? odkaz byl vytvo??en a je nyn?? aktivn??. Zkop??rujte jej n????e a ??i??te ho.",
      "revoke": "Ve??ejn?? p????stupn?? odkaz ???{displayName}??? byl ??sp????n?? odebr??n. Pomoc?? tohoto odkazu nebudou p??ijata ????dn?? dal???? pod??n??."
    }
  },
  "de": {
    "action": {
      "create": "??ffentlichen Zugangslink erstellen"
    },
    "heading": [
      {
        "full": "Jeder mit einem ??ffentlichen Zugangslink kann dieses Formular in einem Webbrowser ausf??llen. Sie k??nnen mehrere Links erstellen, um z.B. unterschiedliche Verteilungen des Formulars nachzuverfolgen oder zu begrenzen wie lange eine spezifische Gruppe von Menschen Zugriff auf das Formular hat. Diese Links funktionieren nur, wenn das Formular den {state} Offen hat.",
        "state": "Status"
      },
      {
        "full": "??ffentliche Links sind f??r das Selbstbeantworten gedacht. Wenn Sie mit Datensammlern arbeiten, die dasselbe Formular mehrfach ??bermitteln m??ssen, {clickHere} f??r andere Optionen.",
        "clickHere": "klicken Sie hier"
      }
    ],
    "emptyTable": "Es gibt keine ??ffentlichen Zugangslinks f??r dieses Formular.",
    "alert": {
      "create": "Ihr ??ffentlicher Zugangslink wurde erfolgreich erstellt und ist jetzt live. Kopieren Sie ihn unten, um ihn zu verteilen.",
      "revoke": "Der ??ffentliche Zugangslink \"{displayName}\" wurde erfolgreich widerrufen. Keine weiteren ??bermittlungen ??ber diesen Link werden akzeptiert."
    }
  },
  "es": {
    "action": {
      "create": "Crear enlace de acceso p??blico"
    },
    "heading": [
      {
        "full": "Cualquier persona con un enlace de acceso p??blico puede rellenar este formulario en un navegador web. Puede crear m??ltiples enlaces para rastrear diferentes distribuciones del formulario, para limitar el tiempo que un grupo espec??fico de personas tiene acceso al Formulario, y m??s. Estos enlaces s??lo funcionar??n si el formulario est?? en el {state} abierto.",
        "state": "estado"
      },
      {
        "full": "Los enlaces p??blicos est??n pensados para autoinformarse. Si usted se encuentra trabajando con recolectores de datos que necesitan enviar el mismo formulario varias veces, {clickHere} para otras opciones.",
        "clickHere": "haga clic aqu??"
      }
    ],
    "emptyTable": "No hay enlaces de acceso p??blico para este formulario.",
    "alert": {
      "create": "??Operaci??n exitosa! Su enlace de acceso p??blico ha sido creado y est?? ahora en vivo. C??pialo abajo para distribuirlo.",
      "revoke": "El enlace de acceso p??blico \"{displayName}\" fue revocado con ??xito. No se aceptar??n m??s env??os a trav??s de este enlace."
    }
  },
  "fr": {
    "action": {
      "create": "Cr??er lien d'acc??s public"
    },
    "heading": [
      {
        "full": "N'importe qui avec ce lien pourra remplir ce formulaire depuis leur navigateur. Vous pouvez cr??er plusieurs liens pour suivre d'o?? viennent vos r??pondants, pour limiter le temps d'acc??s au formulaire et ainsi de suite. Ces liens ne fonctionneront que si le formulaire et dans {state} \"ouvert\".",
        "state": "l'??tat"
      },
      {
        "full": "Les liens d'acc??s public sont con??us pour l'auto-??valuation. Si vous travaillez avec des collecteurs de donn??es qui doivent soumettre le m??me formulaire plusieurs fois, {clickHere} pour d'autres m??thodes d'envoi de donn??es.",
        "clickHere": "cliquez ici"
      }
    ],
    "emptyTable": "Il n'y a aucun lien d'acc??s public pour ce formulaire.",
    "alert": {
      "create": "Succ??s! Votre lien d'acc??s public a ??t?? cr???? et est d??sormais accessible. Copiez le ci-dessous pour le distribuer.",
      "revoke": "Le lien d'acc??s public \"{displayName}\" a ??t?? r??voqu?? avec succ??s. Aucune soumission ne sera accept??e depuis ce lien."
    }
  },
  "id": {
    "action": {
      "create": "Buat Tautan Akses Publik"
    },
    "heading": [
      {
        "full": "Siapapun dengan Tautan Akses Publik dapat mengisi formulir lewat web browser. Anda dapat membuat beberapa tautan untuk melacak pembagian formulir, membatasi seberapa lama kelompok tertentu memiliki akses formulir, dan banyak lagi. Tautan-tautan ini hanya akan berfungsi jika form berada dalam {state} Terbuka.",
        "state": "status"
      },
      {
        "full": "Tautan Publik dimaksudkan untuk laporan pribadi. Apabila Anda bekerja dengan pengumpul data yang butuh mengirimkan beberapa data dalam satu formulir yang sama, {clickHere} untuk pilihan lain.",
        "clickHere": "klik di sini"
      }
    ],
    "emptyTable": "Tidak ada Tautan Akses Publik untuk formulir ini.",
    "alert": {
      "create": "Berhasil! Tautan Akses Publik telah dibuat dan aktif. Salin tautan di bawah untuk disebar.",
      "revoke": "Tautan Akses Publik \"{displayName}\" telah berhasil dicabut. Tidak akan ada lagi kiriman data baru yang diterima lewat tautan ini."
    }
  },
  "it": {
    "action": {
      "create": "Crea Link con accesso pubblico"
    },
    "heading": [
      {
        "full": "Chiunque disponga di un collegamento di accesso pubblico pu?? compilare questo formulario in un browser web. Puoi creare pi?? collegamenti per tenere traccia delle diverse distribuzioni del formulario, per limitare la durata dell'accesso al formulario da parte di un gruppo specifico di persone e altro ancora. Questi collegamenti funzioneranno solo se il formulario ?? nello {state} \"aperto\".",
        "state": "stato"
      },
      {
        "full": "I link pubblici sono intesi per l'auto-riempimento. Se stai lavorando con raccoglitori di dati che devono inviare lo stesso formulario pi?? volte, {clickHere} per altre opzioni.",
        "clickHere": "clicca qui"
      }
    ],
    "emptyTable": "Non ci sono link di accesso pubblico per questo formulario.",
    "alert": {
      "create": "Riuscito! Il tuo link di accesso pubblico ?? stato creato ed ?? ora attivo. Copialo qui sotto per distribuirlo.",
      "revoke": "Il link di accesso pubblico \"{displayName}\" ?? stato revocato con successo. Non saranno accettati ulteriori Invii utilizzando questo Link."
    }
  },
  "ja": {
    "action": {
      "create": "??????????????????????????????"
    },
    "heading": [
      {
        "full": "?????????????????????????????????????????????????????????Web???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????{state}????????????????????????????????????",
        "state": "???????????????"
      },
      {
        "full": "?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????{clickHere}?????????????????????????????????????????????",
        "clickHere": "????????????????????????"
      }
    ],
    "emptyTable": "???????????????????????????????????????????????????????????????",
    "alert": {
      "create": "??????????????????????????????????????????????????????????????????????????? ?????????????????????????????????????????????",
      "revoke": "?????????????????????\"{displayName}\"?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
    }
  },
  "sw": {
    "action": {
      "create": "Unda Kiungo cha Ufikiaji wa Umma"
    },
    "heading": [
      {
        "full": "Mtu yeyote aliye na Kiungo cha Ufikiaji wa Umma anaweza kujaza Fomu hii katika kivinjari. Unaweza kuunda Viungo vingi ili kufuatilia usambazaji tofauti wa Fomu, kuweka kikomo muda ambao kikundi mahususi cha watu kinaweza kufikia Fomu, na zaidi. Viungo hivi vitafanya kazi ikiwa tu Fomu iko katika {state} la Wazi.",
        "state": "Hali"
      },
      {
        "full": "Viungo vya Umma vimekusudiwa kujiripoti. Ikiwa unafanya kazi na wakusanyaji data ambao wanahitaji kuwasilisha Fomu sawa mara nyingi, {clickHere} kwa chaguo zingine.",
        "clickHere": "Bonyeza hapa"
      }
    ],
    "emptyTable": "Hakuna Viungo vya Ufikiaji wa Umma vya Fomu hii.",
    "alert": {
      "create": "Mafanikio! Kiungo chako cha Kufikia Umma kimeundwa na sasa kinapatikana. Nakili hapa chini ili kuisambaza.",
      "revoke": "Kiungo cha Ufikiaji wa Umma \"{displayName}\" kimebatilishwa. Hakuna Mawasilisho Zaidi yatakubaliwa kwa kutumia Kiungo hiki."
    }
  }
}
</i18n>

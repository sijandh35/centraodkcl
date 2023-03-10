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
  <modal id="project-enable-encryption" :state="state"
    :hideable="!awaitingResponse" backdrop
    @hide="$emit(success ? 'success' : 'hide')">
    <template #title>{{ $t('title') }}</template>
    <template #body>
      <template v-if="step === 0">
        <div class="modal-introduction">
          <div class="info-block">
            <p>{{ $t('steps[0].introduction[0][0]') }}</p>
            <div class="info-item">
              <span class="icon-check"></span>
              <p>{{ $t('steps[0].introduction[0][1]') }}</p>
            </div>
            <div class="info-item">
              <span class="icon-check"></span>
              <p>{{ $t('steps[0].introduction[0][2]') }}</p>
            </div>
            <div class="info-item">
              <span class="icon-circle-o"></span>
              <i18n-t tag="p" keypath="steps[0].introduction[0][3][0]">
                <template #submission>
                  <code>&lt;submission&gt;</code>
                </template>
              </i18n-t>
              <i18n-t tag="p" keypath="steps[0].introduction[0][3][1]">
                <template #base64RsaPublicKey>
                  <code>base64RsaPublicKey</code>
                </template>
              </i18n-t>
            </div>
            <div class="info-item">
              <span class="icon-close"></span>
              <p>{{ $t('steps[0].introduction[0][4]') }}</p>
            </div>
            <div class="info-item">
              <span class="icon-close"></span>
              <p>{{ $t('steps[0].introduction[0][5]') }}</p>
            </div>
            <div class="info-item">
              <span class="icon-close"></span>
              <p>{{ $t('steps[0].introduction[0][6]') }}</p>
            </div>
            <div class="info-item">
              <span class="icon-close"></span>
              <p>{{ $t('steps[0].introduction[0][7]') }}</p>
            </div>
          </div>
          <div class="info-block">
            <p>{{ $t('steps[0].introduction[1][0]') }}</p>
            <div class="info-item">
              <span class="icon-circle-o"></span>
              <p>{{ $t('steps[0].introduction[1][1][0]') }}</p>
              <p>{{ $t('steps[0].introduction[1][1][1]') }}</p>
            </div>
            <div class="info-item">
              <span class="icon-close"></span>
              <p>{{ $t('steps[0].introduction[1][2][0]') }}</p>
              <p>{{ $t('steps[0].introduction[1][2][1]') }}</p>
            </div>
          </div>
          <i18n-t tag="p" keypath="steps[0].introduction[2].full">
            <template #here>
              <doc-link to="central-encryption/">{{ $t('steps[0].introduction[2].here') }}</doc-link>
            </template>
          </i18n-t>
        </div>
        <div class="modal-actions">
          <button type="button" class="btn btn-primary" @click="moveToForm">
            {{ $t('action.next') }}
          </button>
          <button type="button" class="btn btn-link" @click="$emit('hide')">
            {{ $t('action.neverMind') }}
          </button>
        </div>
      </template>
      <template v-else-if="step === 1">
        <div class="modal-introduction">
          <p>{{ $t('steps[1].introduction[0]') }}</p>
          <i18n-t tag="p" keypath="steps[1].introduction[1].full">
            <template #no>
              <strong>{{ $t('steps[1].introduction[1].no') }}</strong>
            </template>
          </i18n-t>
        </div>
        <form @submit.prevent="submit">
          <form-group ref="passphrase" v-model="passphrase"
            :placeholder="$t('field.passphrase')" required
            autocomplete="new-password"/>
          <form-group v-model="hint" :placeholder="$t('field.hint')"
            autocomplete="off"/>
          <div class="modal-actions">
            <button type="submit" class="btn btn-danger"
              :disabled="awaitingResponse">
              {{ $t('action.finish') }} <spinner :state="awaitingResponse"/>
            </button>
            <button type="button" class="btn btn-link"
              :disabled="awaitingResponse" @click="$emit('hide')">
              {{ $t('action.neverMind') }}
            </button>
          </div>
        </form>
      </template>
      <template v-else>
        <p id="project-enable-encryption-success-icon-container">
          <span class="icon-check-circle"></span>
        </p>
        <p class="modal-introduction">
          <strong>{{ $t('common.success') }}</strong>
          <sentence-separator/>
          <span>{{ $t('success[0]') }}</span>
        </p>
        <div class="modal-actions">
          <button type="button" class="btn btn-primary"
            @click="$emit('success')">
            {{ $t('action.done') }}
          </button>
        </div>
      </template>
    </template>
  </modal>
</template>

<script>
import DocLink from '../doc-link.vue';
import FormGroup from '../form-group.vue';
import Modal from '../modal.vue';
import SentenceSeparator from '../sentence-separator.vue';
import Spinner from '../spinner.vue';

import request from '../../mixins/request';
import { apiPaths } from '../../util/request';
import { noop } from '../../util/util';
import { useRequestData } from '../../request-data';

export default {
  name: 'ProjectEnableEncryption',
  components: { DocLink, FormGroup, Modal, SentenceSeparator, Spinner },
  mixins: [request()],
  inject: ['alert'],
  props: {
    state: {
      type: Boolean,
      default: false
    }
  },
  emits: ['hide', 'success'],
  setup() {
    const { project } = useRequestData();
    return { project };
  },
  data() {
    return {
      awaitingResponse: false,
      // The step in the wizard
      step: 0,
      passphrase: '',
      hint: '',
      success: false
    };
  },
  watch: {
    state() {
      if (this.state) return;
      this.step = 0;
      this.passphrase = '';
      this.hint = '';
      this.success = false;
    }
  },
  methods: {
    moveToForm() {
      this.step += 1;
      this.$nextTick(() => {
        this.$refs.passphrase.focus();
      });
    },
    submit() {
      if (this.passphrase.length < 10) {
        this.alert.danger(this.$t('alert.passphraseTooShort'));
        return;
      }

      const data = { passphrase: this.passphrase };
      if (this.hint !== '') data.hint = this.hint;
      this.post(apiPaths.projectKey(this.project.id), data)
        .then(() => {
          this.alert.blank();
          this.step += 1;
          this.success = true;
        })
        .catch(noop);
    }
  }
};
</script>

<style lang="scss">
@import '../../assets/scss/variables';

#project-enable-encryption {
  .info-block {
    margin-bottom: 10px;
  }

  .info-item {
    position: relative;

    span {
      left: 3px;
      position: absolute;

      &.icon-check {
        color: $color-success;
      }

      &.icon-close {
        color: $color-danger;
      }

      &.icon-circle-o {
        color: #999;
        top: 1px;
      }
    }

    p {
      margin-bottom: 6px;
      margin-left: 21px;

      + p {
        font-size: 12px;
        margin-top: -3px;
      }

      code {
        background-color: transparent;
        color: $color-text;
        padding: 0;
      }
    }
  }
}

#project-enable-encryption-success-icon-container {
  color: $color-success;
  font-size: 84px;
  line-height: 1;
  text-align: center;
}
</style>

<i18n lang="json5">
{
  "en": {
    // This is the title at the top of a pop-up.
    "title": "Enable Encryption",
    "steps": [
      {
        "introduction": [
          [
            "If you enable encryption, the following things will happen:",
            "Finalized Submission data will be encrypted on mobile devices.",
            "Submission data at rest will be encrypted on the Central server.",
            [
              // {submission} will have the text "<submission>", which is XML
              // and will not be translated.
              "Forms configured with manual {submission} keys will continue to use those keys, and must be manually decrypted.",
              // {base64RsaPublicKey} will have the text "base64RsaPublicKey",
              // which is code and will not be translated.
              "To use the automatic Central encryption process on these Forms, remove the {base64RsaPublicKey} configuration."
            ],
            "You will no longer be able to preview Submission data online.",
            "You will no longer be able to connect to data over OData.",
            "You will no longer be able to edit Submissions in your web browser.",
            "New Submissions will no longer be processed into Entities."
          ],
          [
            "In addition, the following are true in this version of ODK Central:",
            [
              "Existing Submissions will remain unencrypted.",
              "In a future version, you will have the option to encrypt existing data."
            ],
            [
              "Encryption cannot be turned off once enabled.",
              "In a future version, you will be able to disable encryption, which will decrypt your data. This will be true even if you enable encryption now."
            ]
          ],
          {
            "full": "You can learn more about encryption {here}. If this sounds like something you want, press Next to proceed.",
            "here": "here"
          }
        ]
      },
      {
        "introduction": [
          "First, you will need to choose a passphrase. This passphrase will be required to decrypt your Submissions. For your privacy, the server will not remember this passphrase: only people with the passphrase will be able to decrypt and read your Submission data.",
          {
            "full": "If you lose the passphrase, there is {no} way to recover it or your data!",
            "no": "no"
          }
        ]
      }
    ],
    "success": [
      "Encryption has been configured for this Project. Any mobile devices will have to fetch or refetch the latest Forms for encryption to take place."
    ],
    "field": {
      "hint": "Passphrase hint (optional)"
    },
    "alert": {
      "passphraseTooShort": "Please input a passphrase at least 10 characters long."
    }
  }
}
</i18n>

<!-- Autogenerated by destructure.js -->
<i18n>
{
  "cs": {
    "title": "Povolit ??ifrov??n??",
    "steps": [
      {
        "introduction": [
          [
            "Pokud povol??te ??ifrov??n??, stanou se n??sleduj??c?? v??ci:",
            "Dokon??en?? data k odes??l??n?? budou ??ifrov??na na mobiln??ch za????zen??ch.",
            "Data k odes??l??n?? budou nakonec ??ifrov??na na centr??ln??m serveru.",
            [
              "Formul????e nakonfigurovan?? pomoc?? ru??n?? {submission} kl?????? budou tyto kl????e nad??le pou????vat a mus?? b??t ru??n?? de??ifrov??ny.",
              "Chcete-li pou????t automatick?? proces centr??ln??ho ??ifrov??n?? na t??chto formul??????ch, odeberte konfiguraci {base64RsaPublicKey}."
            ],
            "Ji?? nebudete m??t mo??nost zobrazit online n??hled odeslan??ch ??daj??.",
            "Ji?? se nebudete moci p??ipojit k dat??m p??es OData.",
            "Ve sv??m webov??m prohl????e??i ji?? nebudete moci upravovat p????sp??vky.",
            "Nov?? pod??n?? ji?? nebudou zpracov??v??na do entit."
          ],
          [
            "Krom?? toho plat?? pro tuto verzi ODK Central n??sleduj??c??:",
            [
              "St??vaj??c?? p????sp??vky z??stanou neza??ifrovan??.",
              "V budouc?? verzi budete m??t mo??nost ??ifrovat st??vaj??c?? data."
            ],
            [
              "Po aktivaci nelze ??ifrov??n?? vypnout.",
              "V budouc?? verzi budete moci zak??zat ??ifrov??n??, kter?? de??ifruje va??e data. To bude platit, i kdy?? nyn?? povol??te ??ifrov??n??."
            ]
          ],
          {
            "full": "{here} se dozv??te v??ce o ??ifrov??n??. Pokud to zn?? jako n??co, co chcete, pokra??ujte stisknut??m tla????tka Dal????.",
            "here": "zde"
          }
        ]
      },
      {
        "introduction": [
          "Nejprve si mus??te zvolit p????stupov?? heslo. Toto heslo bude vy??adov??no k de??ifrov??n?? va??ich p????sp??vk??. Server si toto heslo nebude pamatovat z d??vodu va??eho soukrom??: pouze lid?? s p????stupov??m slovem budou moci de??ifrovat a p??e????st va??e ??daje o odesl??n??.",
          {}
        ]
      }
    ],
    "success": [
      "Pro tento projekt bylo nakonfigurov??no ??ifrov??n??. V??echna mobiln?? za????zen?? budou muset na????st nebo znovu na????st nejnov??j???? Formul????e, aby mohlo doj??t k ??ifrov??n??."
    ],
    "field": {
      "hint": "P????stupov?? fr??ze (voliteln??)"
    },
    "alert": {
      "passphraseTooShort": "Zadejte alespo?? 10 znak?? dlouhou p????stupovou fr??zi."
    }
  },
  "de": {
    "title": "Verschl??sselung aktivieren",
    "steps": [
      {
        "introduction": [
          [
            "Wenn Sie die Verschl??sselung aktivieren, wird folgendes geschehen:",
            "Daten von abgeschlossenen ??bermittlungen werden auf den Mobilger??ten verschl??sselt.",
            "??bermittlungen auf dem Central Server werden verschl??sselt.",
            [
              "Formulare, die mit manuellen {submission} Schl??sseln konfiguriert wurden, werden diese Schl??ssel weiterhin verwenden und m??ssen manuell entschl??sselt werden.",
              "Um den automatischen Central-Verschl??sselungsprozess f??r diese Formulare zu nutzen, entfernen Sie die {base64RsaPublicKey} Konfiguration."
            ],
            "Sie werden zuk??nftig ??bermittlungsdaten nicht mehr online als Vorschau sehen k??nnen.",
            "Sie werden sich zuk??nftig nicht mehr ??ber OData mit Daten verbinden k??nnen.",
            "Sie k??nnen ??bermittlungen nicht mehr in ihrem Webbrowser bearbeiten."
          ],
          [
            "Au??erdem treffen die folgenden Punkte in dieser Version von ODK Central zu:",
            [
              "Existierende ??bermittlungen bleiben unverschl??sselt.",
              "In einer zuk??nftigen Version werden Sie die Option haben existierende Daten zu verschl??sseln."
            ],
            [
              "Verschl??sselung kann nicht deaktiviert werden, nachdem es einmal aktiviert wurde.",
              "In einer zuk??nftigen Version werden Sie Verschl??sselung deaktivieren k??nnen, was ihre Daten entschl??sseln wird. Dies trifft auch zu, wenn Sie Verschl??sselung jetzt aktivieren."
            ]
          ],
          {
            "full": "Sie k??nnen {here} mehr ??ber Verschl??sselung erfahren. Wenn sich das wollen, klicken Sie auf Weiter, um fortzufahren.",
            "here": "hier"
          }
        ]
      },
      {
        "introduction": [
          "Zuerst m??ssen Sie eine Passphrase ausw??hlen. Diese wird dann ben??tigt, um Ihre ??bermittlungen zu entschl??sseln. Zum Schutz Ihrer Daten wird diese Passphrase nicht gespeichert. Nur Benutzer, die diese Passphrase wissen, k??nnen Ihre ??bermittlungen entschl??sseln.",
          {}
        ]
      }
    ],
    "success": [
      "F??r dieses Projekt wurde Verschl??sselung aktiviert. Alle Mobilger??te m??ssen die neuesten Formulare herunterladen damit die Verschl??sselung auch durchgef??hrt wird."
    ],
    "field": {
      "hint": "Optionale Merkhilfe f??r die Passphrase"
    },
    "alert": {
      "passphraseTooShort": "Bitte geben Sie eine mindestens 10 Zeichen lange Passphrase ein."
    }
  },
  "es": {
    "title": "Habilitar el cifrado",
    "steps": [
      {
        "introduction": [
          [
            "Si usted habilita el cifrado, ocurrir??n las siguientes cosas:",
            "Los datos de la presentaci??n final ser??n cifrados en los dispositivos m??viles.",
            "Los datos de la presentaci??n que se encuentran en reposo ser??n cifrados en el servidor Central.",
            [
              "Los formularios configurados con claves {submission} manuales continuar??n utilizando esas claves y deben ser descifrados manualmente.",
              "Para utilizar el proceso autom??tico de encriptaci??n de Central en estos formularios, remueva la configuraci??n {base64RsaPublicKey}."
            ],
            "Ya no podr?? obtener una vista previa de los datos de la presentaci??n en l??nea",
            "Ya no podr?? conectarse a los datos a trav??s de Odata.",
            "Ya no podr?? editar env??os en su navegador web."
          ],
          [
            "Adem??s, lo siguiente es cierto en esta versi??n de ODK Central:",
            [
              "Los env??os existentes permanecer??n sin cifrar.",
              "En una futura versi??n, tendr?? la opci??n de cifrar los datos existentes."
            ],
            [
              "El cifrado no podr?? ser desactivado una vez activado.",
              "En una versi??n futura, usted podr?? desactivar el cifrado, que permitir?? descifrar sus datos. Esto ser?? efectivo incluso si habilita el cifrado ahora."
            ]
          ],
          {
            "full": "Puede obtener m??s informaci??n sobre el cifrado {here}. Si esto suena como algo que desea hacer, presione siguiente para continuar.",
            "here": "aqu??"
          }
        ]
      },
      {
        "introduction": [
          "En primer lugar, necesitar?? seleccionar una frase de contrase??a. Esta frase ser?? requerida para descifrar sus env??os. Por su privacidad, el servidor no recordar?? esta frase: solo las personas con la frase de contrase??a podr??n descifrar y leer los datos de sus env??os.",
          {}
        ]
      }
    ],
    "success": [
      "El cifrado ha sido configurado para este proyecto. Cualquier dispositivo m??vil tendr?? que buscar o volver a buscar los ??ltimos formularios para que el cifrado se lleve a cabo."
    ],
    "field": {
      "hint": "Sugerencia de frase de contrase??a (opcional)"
    },
    "alert": {
      "passphraseTooShort": "Ingrese una frase de contrase??a de al menos 10 caracteres."
    }
  },
  "fr": {
    "title": "Activer le chiffrement",
    "steps": [
      {
        "introduction": [
          [
            "Si vous activez le chiffrement, les choses suivantes vont se produire :",
            "Les donn??es de soumission finalis??es seront chiffr??es sur les appareils mobiles.",
            "Les donn??es de soumission au repos seront chiffr??es sur le serveur de Central.",
            [
              "Les formulaires configur??s avec des cl??s {submission} manuelles continueront ?? utiliser ces cl??s et devront ??tre d??chiffr??s manuellement.",
              "Pour utiliser le chiffrement automatique de Central pour ce formulaire, supprimez la configuration {base64RsaPublicKey}."
            ],
            "Vous ne pourrez plus pr??visualiser les donn??es de soumission en ligne.",
            "Vous ne pourrez plus vous connecter aux donn??es via OData.",
            "Vous ne serez plus en mesure d'??diter les soumissions dans votre navigateur."
          ],
          [
            "En outre, ce qui suit est vrai dans cette version d'ODK Central :",
            [
              "Les soumissions existantes resteront non chiffr??es.",
              "Dans une prochaine version, vous aurez la possibilit?? de chiffrer les donn??es existantes."
            ],
            [
              "Le chiffrement ne peut pas ??tre d??sactiv?? une fois qu'il est activ??.",
              "Dans une prochaine version, vous pourrez d??sactiver le chiffrement, ce qui permettra de d??chiffrer vos donn??es. Ce sera le cas m??me si vous activez le chiffrement maintenant."
            ]
          ],
          {
            "full": "Vous pouvez en apprendre plus sur le chiffrement en cliquant {here}. Si il vous semble ??tre utile, cliquez sur Suivant pour proc??der.",
            "here": "ici"
          }
        ]
      },
      {
        "introduction": [
          "Tout d'abord vous devrez choisir une phrase de passe. Elle sera requise pour d??chiffrer vos donn??es soumises. Pour respecter votre vie priv??e, le serveur ne retiendra pas cette phrase de passe : seules les personnes disposant de la phrase de passe pourront d??chiffrer et lire vos donn??es envoy??es.",
          {}
        ]
      }
    ],
    "success": [
      "Le chiffrement a ??t?? configur?? pour ce projet. Tous les appareils mobiles devront r??cup??rer ou rer??cup??rer les derniers formulaires pour que le chiffrement puisse avoir lieu."
    ],
    "field": {
      "hint": "Indice de phrase de passe (optionnel)"
    },
    "alert": {
      "passphraseTooShort": "Merci de renseigner une phrase secr??te d'au moins 10 caract??res."
    }
  },
  "id": {
    "title": "Izinkan Enkripsi",
    "steps": [
      {
        "introduction": [
          [
            "Apabila Anda mengizinkan enkripsi, hal-hal berikut akan terjadi:",
            "Kiriman data yang sudah difinalisasi akan dienkripsi di perangkat seluler.",
            "Kiriman data lain akan dienkripsi di server Central.",
            [
              "Formulir yang dikonfigurasi dengan kunci {submission} manual akan tetap menggunakan kunci-kunci tersebut dan harus didekripsi secara manual.",
              "Untuk menggunakan proses enkripsi otomatis Central pada formulir ini, hapus konfigurasi {base64RsaPublicKey}."
            ],
            "Anda tidak akan bisa melihat pratinjau kiriman data secara online.",
            "Anda tidak akan bisa menghubungkan data lewat OData.",
            "Anda tidak akan bisa lagi mengubah kiriman di situs web Anda."
          ],
          [
            "Sebagai tambahan, berikut adalah yang benar ada pada versi ODK Central ini:",
            [
              "Kiriman data yang sudah ada akan tetap tidak terenkripsi.",
              "Pada versi yang akan datang, Anda akan memiliki pilihan untuk mengenkripsi data yang sudah ada."
            ],
            [
              "Enkripsi tidak dapat dimatikan setelah diizinkan.",
              "Pada versi yang akan datang, Anda akan bisa menonaktifkan enkripsi, yang mana akan mendekripsi data Anda. Ini akan terjadi bahkan apabila Anda mengizinkan enkripsi sekarang."
            ]
          ],
          {
            "full": "Anda dapat mempelajari lebih banyak tentang enkripsi {here}. Apabila ini terdengar seperti sesuatu yang Anda inginkan, tekan \"Berikutnya\" untuk melanjutkan.",
            "here": "di sini"
          }
        ]
      },
      {
        "introduction": [
          "Pertama-tama, Anda harus memilih sebuah frasa sandi. Frasa sandi ini akan dibutuhkan untuk mendekripsi kiriman data Anda. Demi pribasi Anda, server tidak akan bisa mengingat frasa sandi ini: hanya orang yang mengetahui frasa sandi yang dapat mendekripsi dan membaca kiriman data Anda.",
          {}
        ]
      }
    ],
    "field": {
      "hint": "Petunjuk frasa sandi (opsional)"
    }
  },
  "it": {
    "title": "Abilita crittografia",
    "steps": [
      {
        "introduction": [
          [
            "Se abiliti la crittografia, accadranno le seguenti cose:",
            "I dati dell'Invio finalizzato verranno crittografati sui dispositivi mobili.",
            "I dati inattivi di invio verranno crittografati sul server Central.",
            [
              "I formulari configurati con chiavi {submission} manuali continueranno a utilizzare tali chiavi e devono essere decrittografati manualmente.",
              "Per utilizzare il processo di crittografia centrale automatica su questi formulari, rimuovere la configurazione {base64RsaPublicKey}."
            ],
            "Non sarai pi?? in grado di visualizzare in anteprima i dati di invio online.",
            "Non sarai pi?? in grado di connetterti ai dati tramite OData.",
            "Non sarai pi?? in grado di modificare gli Invii nel tuo browser web."
          ],
          [
            "Inoltre, quanto segue ?? reale in questa versione di ODK Central:",
            [
              "Gli invii esistenti rimarranno non crittografati.",
              "In una versione futura, avrai la possibilit?? di crittografare i dati esistenti."
            ],
            [
              "La crittografia non pu?? essere disattivata una volta abilitata.",
              "In una versione futura, sarai in grado di disabilitare la crittografia, che decrittografer?? i tuoi dati. Questo sar?? possibile anche se abiliti la crittografia ora."
            ]
          ],
          {
            "full": "Puoi imparare di pi?? sulla crittografia {here}. Se questo ?? quello che desideri, premi Avanti per procedere.",
            "here": "qui"
          }
        ]
      },
      {
        "introduction": [
          "Innanzitutto, dovrai scegliere una passphrase. Questa passphrase sar?? necessaria per decifrare i tuoi Invii. Per la tua privacy, il server non ricorder?? questa passphrase: solo le persone con la passphrase saranno in grado di decifrare e leggere i tuoi dati di invio.",
          {}
        ]
      }
    ],
    "success": [
      "La crittografia ?? stata configurata per questo progetto. Tutti i dispositivi mobili dovranno prendere o recuperare i formulari pi?? recenti affinch?? la crittografia possa avvenire."
    ],
    "field": {
      "hint": "Suggerimento per la passphrase (opzionale)"
    },
    "alert": {
      "passphraseTooShort": "Inserisci una passphrase di almeno 10 caratteri."
    }
  },
  "ja": {
    "title": "???????????????????????????",
    "steps": [
      {
        "introduction": [
          [
            "???????????????????????????????????????????????????????????????",
            "?????????????????????????????????????????????????????????????????????????????????",
            "????????????????????????Central???????????????????????????????????????",
            [
              "?????????{submission}?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
              "???????????????????????????Central????????????????????????????????????????????????{base64RsaPublicKey}????????????????????????????????????"
            ],
            "????????????????????????????????????????????????????????????????????????????????????????????????",
            "OData??????????????????????????????????????????????????????",
            "???????????????????????????Web????????????????????????????????????????????????"
          ],
          [
            "?????????????????????????????????ODK Central?????????????????????????????????????????????",
            [
              "???????????????????????????????????????????????????????????????????????????",
              "?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
            ],
            [
              "??????????????????????????????????????????????????????????????????????????????",
              "???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????"
            ]
          ],
          {
            "full": "?????????????????????????????????{here}????????????????????????????????????????????????????????????????????????????????????????????????????????????",
            "here": "?????????"
          }
        ]
      },
      {
        "introduction": [
          "???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????",
          {}
        ]
      }
    ],
    "field": {
      "hint": "??????????????????????????????????????????"
    },
    "alert": {
      "passphraseTooShort": "?????????????????????10??????????????????????????????????????????"
    }
  },
  "sw": {
    "title": "Washa Usimbaji",
    "steps": [
      {
        "introduction": [
          [
            "Ukiwezesha usimbaji fiche, mambo yafuatayo yatafanyika:",
            "Data iliyokamilishwa ya Uwasilishaji itasimbwa kwa njia fiche kwenye vifaa vya mkononi.",
            "Data ya uwasilishaji imesalia itasimbwa kwa njia fiche kwenye seva ya Central",
            [
              "Fomu zilizosanidiwa kwa funguo za mwongozo za {submission} zitaendelea kutumia funguo hizo, na lazima zisimbuwe wewe mwenyewe",
              "li kutumia mchakato wa usimbaji fiche wa Kati kiotomatiki kwenye Fomu hizi, ondoa usanidi wa {base64RsaPublicKey}."
            ],
            "Hutaweza tena kuhakiki data ya Wasilisho mtandaoni.",
            "Hutaweza tena kuunganisha kwa data kupitia OData.",
            "Hutaweza tena kuhariri Mawasilisho katika kivinjari chako cha wavuti."
          ],
          [
            "kwa kuongeza, yafuatayo ni kweli katika toleo hili la ODK Central:",
            [
              "Mawasilisho yaliyopo yatasalia kuwa hayajasimbwa.",
              "katika toleo la baadaye, utakuwa na chaguo la kusimba data iliyopo"
            ],
            [
              "Usimbaji fiche hauwezi kuzimwa baada ya kuwezeshwa",
              "Katika toleo la baadaye, utaweza kuzima usimbaji fiche, ambao utaondoa usimbaji fiche wa data yako. Hii itakuwa kweli hata ukiwezesha usimbaji fiche sasa."
            ]
          ],
          {
            "full": "Unaweza kupata maelezo zaidi kuhusu usimbaji fiche {here}. Ikiwa hii inaonekana kama kitu unachotaka, bonyeza Ifuatayo ili kuendelea.",
            "here": "hapa"
          }
        ]
      },
      {
        "introduction": [
          "Kwanza, utahitaji kuchagua neno la siri. Kaulisiri hii itahitajika ili kusimbua Mawasilisho yako. Kwa faragha yako, seva haitakumbuka kaulisiri hii: watu walio na kaulisiri pekee wataweza kusimbua na kusoma data yako ya Wasilisho.",
          {}
        ]
      }
    ],
    "field": {
      "hint": "Kidokezo cha kaulisiri (si lazima)"
    },
    "alert": {
      "passphraseTooShort": "Tafadhali weka kaulisiri yenye urefu wa angalau vibambo 10."
    }
  }
}
</i18n>

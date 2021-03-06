<template>
    <b-modal
        id="type"
        ref="modalComponent"
        centered
        no-close-on-esc
        no-close-on-backdrop
        hide-header-close
        hide-footer
        :title="$t('operations:rawsig.title')"
    >
        <table
            v-b-tooltip.hover
            :title="$t('operations:rawsig.request',
                { appName: incoming.appName,
                  origin: incoming.origin,
                  chain: getChainLabel(incoming.chain),
                  accountName: incoming.account_id
                }
            )"
            class="table small table-striped table-sm"
        >
            <tbody>
                <tr>
                    <td class="text-left">
                        Origin
                    </td>
                    <td class="text-right">
                        {{incoming.origin}}
                    </td>
                </tr>
                <tr>
                    <td class="text-left">
                        App
                    </td>
                    <td class="text-right">
                        {{incoming.appName}}
                    </td>
                </tr>
                <tr>
                    <td class="text-left">
                        Account
                    </td>
                    <td class="text-right">
                        {{getChainLabel(incoming.chain) + ":" + (incoming.account_name ? incoming.account_name : incoming.account_id)}}
                    </td>
                </tr>
                <tr>
                    <td class="text-left">
                        Action
                    </td>
                    <td class="text-right">
                        {{getRequestType() ? $t('operations:rawsig.sign_btn') : $t('operations:rawsig.sign_and_broadcast_btn') }}
                    </td>
                </tr>
            </tbody>
        </table>
        <p class="mb-1 font-weight-bold small">
            {{ $t('operations:general.content') }}
        </p>
        <div
            v-if="!!incoming.visualized"
            class="text-left custom-content"
            v-html="incoming.visualized"
        />
        <pre
            v-else-if="!!incoming.params"
            class="text-left custom-content"
        >
<code>
{
{{ incoming.params }}
}
</code>
        </pre>
        {{ $t('operations:rawsig.request_cta') }}
        <b-btn
            class="mt-3"
            variant="success"
            block
            @click="_clickedAllow"
        >
            {{ getRequestType() == "sign" ? $t('operations:rawsig.sign_btn') : $t('operations:rawsig.sign_and_broadcast_btn') }}
        </b-btn>
        <b-btn
            class="mt-1"
            variant="danger"
            block
            @click="_clickedDeny"
        >
            {{ $t('operations:rawsig.reject_btn') }}
        </b-btn>
    </b-modal>
</template>
<script>
    import AbstractPopup from "./abstractpopup";
    import RendererLogger from "../../lib/RendererLogger";
    import getBlockchain from "../../lib/blockchains/blockchainFactory";
    import {getKey} from '../../lib/SecureRemote';
    import {formatChain} from "../../lib/formatter";
    const logger = new RendererLogger();

    export default {
        name: "TransactionRequestPopup",
        extends: AbstractPopup,
        data() {
            return {
                type: "TransactionRequestPopup",
                incoming: {
                    signingAccount: {},
                    visualized: null
                }
            };
        },
        mounted() {
            logger.debug("Tx Popup initialised");
        },
        methods: {
            getChainLabel: function(chain) {
                return formatChain(chain);
            },
            getRequestType: function() {
                if (this.incoming.params && this.incoming.params.length > 0 && this.incoming.params[0] == "sign") {
                    return "sign"
                } else {
                    return "sign_and_broadcast";
                }
            },
            _onShow() {
                let blockchain = getBlockchain(this.incoming.chain);
                blockchain.visualize(this.incoming.params).then(result => {
                    this.incoming.visualized = result;
                    this.incoming = Object.assign({}, this.incoming);
                    blockchain.visualize(this.incoming.account_id).then(account_name => {
                        this.incoming.account_name = account_name;
                        this.incoming = Object.assign({}, this.incoming);
                    });
                })

            },
            getSuccessNotification(result) {
                return {msg: 'Transaction successfully broadcast.', link: '' };
            },
            _execute: async function () {
                let blockchain = getBlockchain(this.incoming.chain);
                if (this.incoming.params[0] == "sign") {
                    let tr = await blockchain.sign(
                        this.incoming.params,
                        await getKey(this.$store.getters['AccountStore/getSigningKey'](this.incoming).keys.active)
                    );
                    return tr.toObject();
                } else if (this.incoming.params[0] == "broadcast") {
                    return await blockchain.broadcast(this.incoming.params);
                } else if (this.incoming.params[0] == "signAndBroadcast") {
                    let transaction = await blockchain.sign(
                        this.incoming.params,
                        await getKey(this.$store.getters['AccountStore/getSigningKey'](this.incoming).keys.active)
                    );
                    return await blockchain.broadcast(transaction);
                }
            }
        }
    };
</script>

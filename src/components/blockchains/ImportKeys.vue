<template>
    <div
        id="step2"
    >
        <h4 class="h4 mt-3 font-weight-bold">
            {{ $t('step_counter',{ 'step_no' : 2}) }}
        </h4>
        <p
            class="mb-2 font-weight-bold"
        >
            {{ $t(accessType == 'account' ? 'account_name' : 'address_name', { 'chain' : selectedChain}) }}
        </p>
        <input
            id="inputAccount"
            v-model="accountname"
            type="text"
            class="form-control mb-3"
            :placeholder="$t(accessType == 'account' ? 'account_name' : 'address_name', { 'chain' : selectedChain})"
            required
        >
        <p class="my-3 font-weight-normal">
            {{ $t('keys_cta') }}
        </p>
        <template v-if="requiredFields.active !== null">
            <p
                class="mb-2 font-weight-bold"
            >
                {{ $t(accessType == 'account' ? 'active_authority' : 'public_authority') }}
            </p>
            <input
                id="inputActive"
                v-model="activepk"
                type="password"
                class="form-control mb-3 small"
                :placeholder="$t(accessType == 'account' ? 'active_authority_placeholder' : 'public_authority_placeholder')"
                required
            >
        </template>
        <template v-if="requiredFields.memo !== null">
            <p class="mb-2 font-weight-bold">
                {{ $t('memo_authority') }}
            </p>
            <input
                id="inputMemo"
                v-model="memopk"
                type="password"
                class="form-control mb-3 small"
                :placeholder="$t('memo_authority_placeholder')"
                required
            >
        </template>
        <template v-if="requiredFields.owner !== null">
            <b-form-checkbox
                id="incOwnerCB"
                v-model="includeOwner"
                value="1"
                unchecked-value="0"
                class="mb-3"
            >
                {{ $t('include_owner_check') }}
            </b-form-checkbox>
            <div v-if="includeOwner==1">
                <input
                    id="inputOwner"
                    v-model="ownerpk"
                    type="password"
                    class="form-control mb-3 small"
                    :placeholder="$t('owner_authority_placeholder')"
                    required
                >
            </div>
        </template>
    </div>

</template>

<script>
    import getBlockchain from "../../lib/blockchains/blockchainFactory";

    export default {
        name: "ImportKeys",
        i18nOptions: { namespaces: "common" },
        props: [ "selectedChain" ],
        data() {
            return {
                accountname: "",
                activepk: "",
                ownerpk: "",
                memopk: "",
                includeOwner: 0,
                accessType: null,
                requiredFields: null
            };
        },
        created() {
            let blockchain = getBlockchain(this.selectedChain);
            this.accessType = blockchain.getAccessType();
            this.requiredFields = blockchain.getSignUpInput();
        },
        methods: {
            _verifyAccount: async function() {
                if (this.accountname == "") {
                    throw {
                        key: "missing_account_error",
                        args: {
                            chain: this.selectedChain
                        }
                    };
                }
                let blockchain = getBlockchain(this.selectedChain);
                let authorities = {};
                if (this.requiredFields.active != null) {
                    authorities.active = this.activepk;
                }
                if (this.requiredFields.memo != null) {
                    authorities.memo = this.memopk;
                }
                if (this.includeOwner == 1 && this.requiredFields.owner != null) {
                    authorities.owner = this.ownerpk;
                }
                let account = await blockchain.verifyAccount(this.accountname, authorities);
                return {
                    account: {
                        accountName: this.accountname,
                        accountID: account.id,
                        chain: this.selectedChain,
                        keys: authorities
                    }
                };
            },
            getAccountEvent: async function() {
                let account = await this._verifyAccount();
                if (this.accountID !== null) {
                    return [account];
                } else {
                    throw "This shouldn't happen!"
                }
            }
        }
    };
</script>
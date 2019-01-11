<template>
    <div class="bottom">
        <input type="file" @change="handleWalletSelect" v-if="step1"/>
        <input type="password" v-model="wallet_pass" v-if="step1"/>
        <button
            class="btn btn-lg btn-primary btn-block"
            type="submit"
            @click="decryptBackup"
             v-if="step1"
        >{{ $t('next_btn') }}</button>
        <div class="import-accounts mt-3" v-if="step2">
            <table class="table small table-striped table-sm">
                <thead>
                    <tr>                    
                        <th rowspan="2" class="align-middle">Account Name</th>
                        <th colspan="2" class="align-middle">Active Authority</th>
                        <th colspan="2" class="align-middle">Owner Authority</th>
                        <th rowspan="2" class="align-middle">Memo</th>
                        <th rowspan="2" class="align-middle">Import?</th>
                    </tr>                
                    <tr>
                        <th class="align-middle">Propose</th>
                        <th class="align-middle">Transact</th>
                        <th class="align-middle">Propose</th>
                        <th class="align-middle">Transact</th>
                    </tr>
                </thead>
                <tbody>
                    <tr
                        v-for="account in accounts" 
                        :key="account.id"
                    >                    
                        <td class="text-center align-middle">{{ account.name }}<br/>({{ account.id }})</td>
                        <td class="text-center align-middle">{{ account.active.canPropose ? 'Y' : 'N' }}</td>
                        <td class="text-center align-middle">{{ account.active.canTransact ? 'Y' : 'N' }}</td>
                        <td class="text-center align-middle">{{ account.owner.canPropose ? 'Y' : 'N' }}</td>
                        <td class="text-center align-middle">{{ account.owner.canTransact ? 'Y' : 'N' }}</td>
                        <td class="text-center align-middle">{{ account.memo.canSend ? 'Y' : 'N' }}</td>
                        <td class="text-center align-middle"><input type="checkbox" :id="account.name" :value="account.id" v-model="picked" /></td>
                    </tr>
                </tbody>
            </table>
        </div>
        <button class="btn btn-lg btn-primary btn-block mt-3"  v-if="step2 && picked.length>0">
            Import Selected
        </button>
        <b-modal
            id="loaderAnim"
            ref="loaderAnimModal"
            centered
            no-close-on-esc
            no-close-on-backdrop
            hide-header
            hide-header-close
            hide-footer
            title="Loading..."
        >
            <div class="lds-roller"><div /><div /><div /><div /><div /><div /><div /><div /></div>
        </b-modal>
        <b-modal
            id="error"
            ref="errorModal"
            centered
            hide-footer
            :title="$t('error_lbl')"
        >
            {{ errorMsg }}
        </b-modal>
        
        <p class="mt-2 mb-2 small">&copy; 2018 BitShares</p>
    </div>
</template>

<script>
    import LangSelect from "./lang-select";
    import {PrivateKey,PublicKey,Aes} from "bitsharesjs";
    import {compress, decompress} from "lzma";    
    import { Apis } from "bitsharesjs-ws";
    import WalletHandler from "../lib/WalletHandler";

    export default {
        name: "ImportBin",
        i18nOptions: { namespaces: "common" },
        components: { LangSelect },
        data() {
            return {
                wallet_file: null,
                wallet_pass: null,
                errorMsg: '',
                step1:true,
                step2:false,
                accounts:[],
                picked:[]
            };
        },
        methods: {
            handleWalletSelect: function(e) {
                this.wallet_file=e.target.files[0];
            },
            decryptBackup: function() {
                this.$refs.loaderAnimModal.show();
                let reader = new FileReader();
                reader.onload = async evt => {
                    let backup_buffer = new Buffer.from(evt.target.result, "binary");                
                    let wh=new WalletHandler(evt.target.result);
                    try {
                        let unlocked=await wh.unlock(this.wallet_pass);
                        if (unlocked) {
                            this.accounts=await wh.lookupAccounts();
                            console.log(this.accounts);
                            this.step1=false;
                            this.step2=true;
                            this.$refs.loaderAnimModal.hide();
                        }
                    }catch(e) {
                        console.log(e);
                    }
                };
                reader.readAsBinaryString(this.wallet_file);
            }
        }
    };
</script>
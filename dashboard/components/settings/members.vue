<script setup lang="ts">
import type { SettingsTemplateEntry } from './Template.vue';

const { projectId, isGuest } = useProject();

definePageMeta({ layout: 'dashboard' });

const columns = [
    { key: 'me', label: '' },
    { key: 'email', label: 'Email' },
    { key: 'name', label: 'Name' },
    { key: 'role', label: 'Role' },
    { key: 'action', label: 'Actions' },
    // { key: 'pending', label: 'Pending' },
]

const { data: members, refresh: refreshMembers } = useFetch('/api/project/members/list', {
    headers: useComputedHeaders({ useSnapshotDates: false })
});

const showAddMember = ref<boolean>(false);

const addMemberEmail = ref<string>("");

async function kickMember(email: string) {

    const sure = confirm('Are you sure to kick ' + email + ' ?');
    if (!sure) return;

    try {

        await $fetch('/api/project/members/kick', {
            method: 'POST',
            ...signHeaders({
                'Content-Type': 'application/json',
                'x-pid': projectId.value ?? ''
            }),
            body: JSON.stringify({ email }),
            onResponseError({ request, response, options }) {
                alert(response.statusText);
            }
        });

        refreshMembers();

    } catch (ex: any) { }



}

async function addMember() {

    if (addMemberEmail.value.length === 0) return;

    try {

        showAddMember.value = false;

        await $fetch('/api/project/members/add', {
            method: 'POST',
            ...signHeaders({
                'Content-Type': 'application/json',
                'x-pid': projectId.value ?? ''
            }),
            body: JSON.stringify({ email: addMemberEmail.value }),
            onResponseError({ request, response, options }) {
                alert(response.statusText);
            }
        });

        addMemberEmail.value = '';

        refreshMembers();

    } catch (ex: any) { }


}

const entries: SettingsTemplateEntry[] = [
    { id: 'add', title: 'Add member', text: 'Add new member to project' },
    { id: 'members', title: 'Members', text: 'Manage members of current project' },
]

</script>

<template>

    <SettingsTemplate :entries="entries">
        <template #add>
            <div v-if="!isGuest" class="flex flex-col">
                <div class="flex gap-4 items-center">
                    <LyxUiInput class="px-4 py-1 w-full" placeholder="User email" v-model="addMemberEmail"></LyxUiInput>
                    <LyxUiButton @click="addMember" type="secondary"> Add </LyxUiButton>
                </div>
                <div class="poppins text-[.8rem] mt-2 text-lyx-text-darker">
                    User should have been registered to Litlyx
                </div>
            </div>
            <div v-if="isGuest" class="text-lyx-text-darker"> Guests cannot add members</div>
        </template>
        <template #members>


            <UTable :rows="members || []" :columns="columns">
                <template #me-data="e">
                    <i v-if="e.row.me" class="far fa-user"></i>
                    <i v-if="!e.row.me"></i>
                </template>

                <template #action-data="e" v-if="!isGuest">
                    <div @click="kickMember(e.row.email)" v-if="e.row.role != 'OWNER'"
                        class="text-red-500 hover:bg-black/20 cursor-pointer outline outline-[1px] outline-red-500 px-3 py-1 rounded-lg text-center">
                        Kick
                    </div>
                </template>

            </UTable>
        </template>
    </SettingsTemplate>

</template>

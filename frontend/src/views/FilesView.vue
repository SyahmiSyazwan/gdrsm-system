<script setup>
    import { ref } from 'vue'
    import { onMounted } from 'vue'
    import DataTable from 'primevue/datatable'
    import Column from 'primevue/column'
    import Button from 'primevue/button'
    import Dialog from 'primevue/dialog'
    import InputText from 'primevue/inputtext'
    import Dropdown from 'primevue/dropdown'
    import Textarea from 'primevue/textarea';
    import Sidebar from '../components/Sidebar.vue'

    // File types dropdown options
    const fileTypes = ref([
        { name: 'PDF', code: 'PDF' },
        { name: 'DOCX', code: 'DOCX' },
        { name: 'PPTX', code: 'PPTX' },
        { name: 'CSV', code: 'CSV' },
        { name: 'Other', code: 'OTHER' }
    ])

    const resources = ref([]);

    const resourceData = ref({
        name: '',
        type: '',
        description: '',
        owner: '',
        link: '',
        session: '',
        semester: '',
        folder: '',
        category: ''
    })

    const showAddResourceDialog = ref(false);

    const files = ref([
        {
            id: 1,
            name: 'Assignment 1',
            type: 'PDF',
            size: '2.5 MB',
            uploadDate: '2023-06-15',
            sharedBy: 'Dr. John Doe',
            shareDate: '2023-06-16'
        },
        {
            id: 2,
            name: 'Lecture Notes',
            type: 'DOCX',
            size: '1.2 MB',
            uploadDate: '2023-06-10',
            sharedBy: 'Prof. Jane Smith',
            shareDate: '2023-06-12'
        },
        {
            id: 3,
            name: 'Machine Learning Dataset',
            type: 'CSV',
            size: '50.7 MB',
            uploadDate: '2023-06-05',
            sharedBy: 'Dr. Alan Turing',
            shareDate: '2023-06-08'
        },
        {
            id: 4,
            name: 'Research Proposal',
            type: 'PPTX',
            size: '15.3 MB',
            uploadDate: '2023-05-30',
            sharedBy: 'Dr. Marie Curie',
            shareDate: '2023-06-02'
        },
        {
            id: 5,
            name: 'Network Security Slides',
            type: 'PDF',
            size: '8.7 MB',
            uploadDate: '2023-06-20',
            sharedBy: 'Prof. Dennis Ritchie',
            shareDate: '2023-06-22'
        },
        {
            id: 6,
            name: 'Database Design Report',
            type: 'DOCX',
            size: '4.5 MB',
            uploadDate: '2023-06-18',
            sharedBy: 'Dr. Grace Hopper',
            shareDate: '2023-06-19'
        },
        {
            id: 7,
            name: 'AI Ethics Paper',
            type: 'PDF',
            size: '6.2 MB',
            uploadDate: '2023-06-12',
            sharedBy: 'Prof. Andrew Ng',
            shareDate: '2023-06-14'
        },
        {
            id: 8,
            name: 'Cloud Computing Notes',
            type: 'PPTX',
            size: '12.1 MB',
            uploadDate: '2023-06-08',
            sharedBy: 'Dr. Werner Vogels',
            shareDate: '2023-06-10'
        },
        {
            id: 9,
            name: 'Software Engineering Guidelines',
            type: 'PDF',
            size: '3.8 MB',
            uploadDate: '2023-06-16',
            sharedBy: 'Prof. Martin Fowler',
            shareDate: '2023-06-17'
        }
    ])

    // Modal state
    const visible = ref(false)
    const isEditMode = ref(false)
    const currentFile = ref({
        name: '',
        type: null,
        size: '',
        googleDriveLink: '',
    })

    // Modal open handlers
    const openAddModal = () => {
        isEditMode.value = false
        currentFile.value = {
            name: '',
            type: null,
            size: '',
            googleDriveLink: '',
        }
        visible.value = true
    }

    const openEditModal = (file) => {
        isEditMode.value = true
        currentFile.value = { ...file }
        visible.value = true
    }

    // Save file handler
    const saveFile = () => {
        if (isEditMode.value) {
            // Update existing file
            const index = files.value.findIndex(f => f.id === currentFile.value.id)
            files.value[index] = {
                ...currentFile.value,
                shareDate: new Date().toISOString().split('T')[0],
                sharedBy: 'Current User' // In real app, use actual logged-in user
            }
        } else {
            // Add new file
            const newFile = {
                ...currentFile.value,
                id: files.value.length + 1,
                uploadDate: new Date().toISOString().split('T')[0],
                shareDate: new Date().toISOString().split('T')[0],
                sharedBy: 'Current User' // In real app, use actual logged-in user
            }
            files.value.push(newFile)
        }
        visible.value = false
    }

    // Delete file handler
    const deleteFile = (file) => {
        files.value = files.value.filter(f => f.id !== file.id)
    }

    const saveResource = async () => {
        const url = 'http://127.0.0.1:3000/api/resources';
        try {
            const response = await fetch('http://127.0.0.1:3000/api/resources', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(resourceData.value)
            });

            if (!response.ok) {
                throw new Error(`Response status: ${response.status}`);
            }

            const data = await response.json();
            console.log('Success:', data);

            await fetchResources();

            resourceData.value = {
                name: '',
                type: '',
                description: '',
                owner: '',
                link: '',
                session: '',
                semester: '',
                folder: '',
                category: ''
            };

            showAddResourceDialog.value = false

        } catch (error) {
            console.error(error.message);
        }
    }

    const fetchResources = async () => {
        const url = 'http://127.0.0.1:3000/api/resources';
        try {
            const response = await fetch(url);
            if (!response.ok) {
                throw new Error(`Response status: ${response.status}`);
            }

            const fetchedResources = await response.json();
            resources.value = fetchedResources;
        } catch (error) {
            console.error(error.message);
        }
    }

    onMounted(fetchResources)
</script>

<template>
    <div class="flex min-h-screen">
        <Sidebar />
        <div class="grow bg-gray-100 p-4">
            <div class="card">
                <div class="flex justify-between mb-4">
                    <h2 class="text-2xl font-bold">File Management</h2>
                    <Button
                        label="Add New Resource"
                        icon="pi pi-plus"
                        @click="showAddResourceDialog = true"
                    />
                </div>

                <DataTable :value="resources" stripedRows>
                    <Column field="name" header="Name"></Column>
                    <Column field="type" header="Type"></Column>
                    <Column field="description" header="Description"></Column>
                    <Column field="owner" header="Owner"></Column>
                    <Column field="link" header="Link"></Column>
                    <Column field="date_created" header="Date Added"></Column>
                    <Column field="session" header="Session"></Column>
                    <Column field="semester" header="Semester"></Column>
                    <Column header="Actions">
                        <template #body="{ data }">
                            <div class="flex gap-2">
                                <Button
                                    icon="pi pi-pencil"
                                    class="p-button-info p-button-sm"
                                    @click="openEditModal(data)"
                                />
                                <Button
                                    icon="pi pi-trash"
                                    class="p-button-danger p-button-sm"
                                    @click="deleteFile(data)"
                                />
                            </div>
                        </template>
                    </Column>
                </DataTable>

                <!-- File Modal -->
                <Dialog
                    v-model:visible="visible"
                    :header="isEditMode ? 'Edit File' : 'Add New File'"
                    modal
                    class="w-full max-w-[500px]"
                >
                    <div class="grid grid-cols-1 gap-4 p-4">
                        <div class="flex flex-col">
                            <label class="mb-2 font-semibold">File Name</label>
                            <InputText
                                v-model="currentFile.name"
                                placeholder="Enter file name"
                                class="w-full"
                            />
                        </div>

                        <div class="flex flex-col">
                            <label class="mb-2 font-semibold">File Type</label>
                            <Dropdown
                                v-model="currentFile.type"
                                :options="fileTypes"
                                optionLabel="name"
                                optionValue="code"
                                placeholder="Select file type"
                                class="w-full"
                            />
                        </div>

                        <div class="flex flex-col">
                            <label class="mb-2 font-semibold">File Size (optional)</label>
                            <InputText
                                v-model="currentFile.size"
                                placeholder="Enter file size (e.g., 2.5 MB)"
                                class="w-full"
                            />
                        </div>

                        <div class="flex flex-col">
                            <label class="mb-2 font-semibold">Google Drive Link</label>
                            <InputText
                                v-model="currentFile.googleDriveLink"
                                placeholder="Paste Google Drive share link"
                                class="w-full"
                            />
                            <small class="text-xs text-gray-500 mt-1">
                                Ensure you've set the correct sharing permissions for the file
                            </small>
                        </div>

                        <div class="flex justify-end mt-4">
                            <Button
                                :label="isEditMode ? 'Update' : 'Add'"
                                @click="saveFile"
                                class="p-button-primary"
                            />
                        </div>
                    </div>
                </Dialog>

                <Dialog v-model:visible="showAddResourceDialog" modal header="Add New Resource" :style="{ width: '30rem' }">
                    <div class="flex items-center gap-4 mb-4">
                        <label for="name" class="font-semibold w-24">Name</label>
                        <InputText id="name" v-model="resourceData.name" class="flex-auto" autocomplete="off" />
                    </div>

                    <div class="flex items-center gap-4 mb-4">
                        <label for="type" class="font-semibold w-24">Type</label>
                        <Dropdown 
                            id="type"
                            v-model="resourceData.type"
                            :options="['File', 'Folder']" 
                            class="flex-auto" 
                            placeholder="Select Type"
                        />
                    </div>

                    <div class="flex items-center gap-4 mb-4">
                        <label for="description" class="font-semibold w-24">Description</label>
                        <Textarea id="description" v-model="resourceData.description" class="flex-auto" rows="3" />
                    </div>

                    <div class="flex items-center gap-4 mb-4">
                        <label for="owner" class="font-semibold w-24">Owner</label>
                        <InputText id="owner" v-model="resourceData.owner" type="email" class="flex-auto" autocomplete="off" />
                    </div>

                    <div class="flex items-center gap-4 mb-4">
                        <label for="link" class="font-semibold w-24">Link</label>
                        <InputText id="link" v-model="resourceData.link" class="flex-auto" autocomplete="off" />
                    </div>

                    <div class="flex items-center gap-4 mb-4">
                        <label for="session" class="font-semibold w-24">Session</label>
                        <Dropdown 
                            id="session"
                            v-model="resourceData.session"
                            :options="['2023/2024', '2024/2025', '2025/2026']" 
                            class="flex-auto" 
                            placeholder="Select Session"
                        />
                    </div>

                    <div class="flex items-center gap-4 mb-4">
                        <label for="semester" class="font-semibold w-24">Semester</label>
                        <Dropdown 
                            id="semester"
                            v-model="resourceData.semester"
                            :options="['1', '2', '3']" 
                            class="flex-auto" 
                            placeholder="Select Semester"
                        />
                    </div>

                    <div class="flex items-center gap-4 mb-4">
                        <label for="folder" class="font-semibold w-24">Folder</label>
                        <Dropdown 
                            id="folder" 
                            v-model="resourceData.folder"
                            :options="['Folder 1', 'Folder 2', 'Folder 3']" 
                            class="flex-auto" 
                            placeholder="Select Folder"
                        />
                    </div>

                    <div class="flex items-center gap-4 mb-8">
                        <label for="category" class="font-semibold w-24">Category</label>
                        <Dropdown 
                            id="category" 
                            v-model="resourceData.category"
                            :options="['Category 1', 'Category 2', 'Category 3']" 
                            class="flex-auto" 
                            placeholder="Select Category"
                        />
                    </div>

                    <div class="flex justify-end gap-2">
                        <Button type="button" label="Cancel" severity="secondary" @click="showAddResourceDialog = false"></Button>
                        <Button type="button" label="Save" @click="saveResource"></Button>
                    </div>
                </Dialog>
            </div>
        </div>
    </div>
</template>

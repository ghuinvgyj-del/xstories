<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>xstories | Narrative Platform</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        primary: '#4f46e5', // Indigo
                        accent: '#f59e0b',  // Sunset Amber
                    }
                }
            }
        }
    </script>
    <style>
        .tab-active { color: #f59e0b; border-bottom: 2px solid #f59e0b; }
        .dark .tab-active { color: #fbbf24; border-bottom: 2px solid #fbbf24; }
        .cat-pill.active { background: #f59e0b !important; color: white; }
        .btn-gradient { background: linear-gradient(135deg, #4f46e5, #f59e0b); }
    </style>
</head>
<body class="bg-slate-50 text-slate-900 min-h-screen dark:bg-slate-950 dark:text-slate-100 transition-colors duration-500">

<div class="max-w-4xl mx-auto px-6 py-12">
    <nav class="flex justify-between items-center mb-12">
        <div>
            <h1 class="text-4xl font-extrabold tracking-tighter mb-1 text-indigo-600 dark:text-indigo-400">xstories</h1>
            <p class="text-slate-500 dark:text-slate-400 font-medium uppercase tracking-widest text-xs">Narrative Platform</p>
        </div>
        <div class="flex items-center gap-4">
            <button onclick="toggleTheme()" class="w-12 h-12 rounded-full bg-slate-200 dark:bg-slate-800 flex items-center justify-center hover:scale-105 transition text-2xl">🌓</button>
            <button onclick="showModal()" class="px-6 py-3 btn-gradient text-white rounded-2xl font-bold shadow-lg hover:shadow-indigo-500/20 transition">New Post</button>
        </div>
    </nav>

    <div class="flex gap-8 border-b border-slate-200 dark:border-slate-800 mb-8 overflow-x-auto pb-1">
        <button onclick="setTab('stories')" id="tab-stories" class="pb-3 text-sm font-bold tracking-wide uppercase tab-active whitespace-nowrap">Stories</button>
        <button onclick="setTab('docs')" id="tab-docs" class="pb-3 text-sm font-bold tracking-wide uppercase whitespace-nowrap">Documents</button>
        <button onclick="setTab('collections')" id="tab-collections" class="pb-3 text-sm font-bold tracking-wide uppercase whitespace-nowrap">Collections</button>
        <button onclick="setTab('tags')" id="tab-tags" class="pb-3 text-sm font-bold tracking-wide uppercase whitespace-nowrap">Tags</button>
    </div>

    <div id="categoryFilterBar" class="flex flex-wrap gap-2 mb-10"></div>
    <div id="main-content" class="space-y-8"></div>
</div>

<div id="modal" class="fixed inset-0 bg-black/60 backdrop-blur-md hidden z-50 flex items-center justify-center p-4">
    <div class="bg-white dark:bg-slate-900 w-full max-w-2xl rounded-3xl p-8 shadow-2xl border border-slate-200 dark:border-slate-800 max-h-[90vh] overflow-y-auto">
        <div class="flex justify-between items-center mb-6">
            <h2 class="text-2xl font-bold">New Post</h2>
            <select id="postType" onchange="updateModalFields()" class="bg-slate-100 dark:bg-slate-800 px-4 py-2 rounded-xl text-sm font-bold border border-slate-200 dark:border-slate-700">
                <option value="story">Story</option>
                <option value="doc">Document</option>
            </select>
        </div>
        <div id="modalFields" class="space-y-5"></div>
        <div class="flex justify-end gap-3 mt-8 pt-6 border-t dark:border-slate-800">
            <button onclick="hideModal()" class="px-6 py-3 font-bold text-slate-500">Cancel</button>
            <button onclick="saveItem()" class="px-8 py-3 btn-gradient text-white rounded-xl font-bold shadow-lg transition">Publish</button>
        </div>
    </div>
</div>

<script>
    let activeTab = 'stories', filterCat = null, filterTag = null, storyBlocks = [{ type: 'text', content: '' }];
    const CATEGORIES = ['Fantasy', 'Adultery', 'Romance', 'Sci-Fi', 'Thriller', 'Horror', 'Incest'];
    let stories = JSON.parse(localStorage.getItem('xstories_data') || '[]');
    let docs = JSON.parse(localStorage.getItem('xdocs_data') || '[]');

    function toggleTheme() { document.documentElement.classList.toggle('dark'); }
    function setTab(tab, tag = null) {
        activeTab = tab; filterCat = null; filterTag = tag;
        document.querySelectorAll('button[id^="tab-"]').forEach(b => b.classList.remove('tab-active'));
        document.getElementById('tab-' + tab).classList.add('tab-active');
        document.getElementById('categoryFilterBar').classList.toggle('hidden', tab === 'tags');
        render();
    }

    function showModal() { storyBlocks = [{ type: 'text', content: '' }]; document.getElementById('modal').classList.remove('hidden'); updateModalFields(); }
    function hideModal() { document.getElementById('modal').classList.add('hidden'); }

    function addBlock(type, index) { 
        storyBlocks.splice(index + 1, 0, { type, content: '', url: '' }); 
        updateModalFields(); 
    }
    function removeBlock(index) { if (storyBlocks.length > 1) { storyBlocks.splice(index, 1); updateModalFields(); } }
    function updateBlock(index, val) { storyBlocks[index].content = val; }
    function handleFile(index, file) {
        const reader = new FileReader();
        reader.onload = (e) => { storyBlocks[index].url = e.target.result; updateModalFields(); };
        reader.readAsDataURL(file);
    }

    function updateModalFields() {
        const type = document.getElementById('postType').value;
        const container = document.getElementById('modalFields');
        if (type === 'story') {
            container.innerHTML = `
                <input type="text" id="title" placeholder="Story Title" class="w-full p-4 rounded-xl border border-slate-200 dark:border-slate-700 bg-slate-50 dark:bg-slate-800 font-medium">
                <input type="text" id="series" placeholder="Collection/Series Name" class="w-full p-4 rounded-xl border border-slate-200 dark:border-slate-700 bg-slate-50 dark:bg-slate-800">
                <div class="flex flex-wrap gap-2" id="catContainer">
                    ${CATEGORIES.map(c => `<button type="button" onclick="this.classList.toggle('active')" class="px-4 py-2 rounded-full text-xs font-bold bg-slate-200 dark:bg-slate-800 cat-pill" data-cat="${c}">${c}</button>`).join('')}
                </div>
                <input type="text" id="hashtags" placeholder="Tags (comma separated)" class="w-full p-4 rounded-xl border border-slate-200 dark:border-slate-700 bg-slate-50 dark:bg-slate-800">
                <div class="space-y-4">
                    ${storyBlocks.map((b, i) => `
                        <div class="relative group p-2 rounded-2xl">
                            <div class="flex gap-2 mb-2 opacity-50 group-hover:opacity-100 transition">
                                <button type="button" onclick="addBlock('text', ${i})" class="text-[9px] font-bold text-indigo-500 uppercase">+ Text</button>
                                <button type="button" onclick="addBlock('image', ${i})" class="text-[9px] font-bold text-indigo-500 uppercase">+ Image</button>
                                <button type="button" onclick="removeBlock(${i})" class="text-[9px] font-bold text-red-500 uppercase ml-auto">Delete</button>
                            </div>
                            <div class="p-4 bg-slate-50 dark:bg-slate-800 rounded-2xl border border-slate-200 dark:border-slate-700">
                                ${b.type === 'text' ? `<textarea oninput="updateBlock(${i}, this.value)" class="w-full min-h-[100px] bg-transparent outline-none" placeholder="Write here...">${b.content}</textarea>` : 
                                  (b.url ? `<img src="${b.url}" class="w-full h-40 object-cover rounded-xl">` : `<input type="file" accept="image/*" onchange="handleFile(${i}, this.files[0])" class="w-full">`)}
                            </div>
                        </div>
                    `).join('')}
                </div>
            `;
        } else {
            container.innerHTML = `<input type="text" id="docTitle" placeholder="Document Title" class="w-full p-4 rounded-xl border border-slate-200 dark:border-slate-700 bg-slate-50 dark:bg-slate-800">`;
        }
    }

    function saveItem() {
        if (document.getElementById('postType').value === 'story') {
            const cats = Array.from(document.querySelectorAll('.cat-pill.active')).map(p => p.dataset.cat);
            stories.unshift({ id: Date.now(), title: document.getElementById('title').value, series: document.getElementById('series').value, cats, tags: document.getElementById('hashtags').value.split(','), blocks: storyBlocks });
            localStorage.setItem('xstories_data', JSON.stringify(stories));
        } else {
            docs.unshift({ id: Date.now(), title: document.getElementById('docTitle').value });
            localStorage.setItem('xdocs_data', JSON.stringify(docs));
        }
        hideModal(); render();
    }

    function render() {
        const content = document.getElementById('main-content');
        const catBar = document.getElementById('categoryFilterBar');
        catBar.innerHTML = `<button onclick="filterCat=null; render()" class="px-5 py-2 rounded-full text-xs font-bold ${!filterCat ? 'bg-indigo-600 text-white' : 'bg-slate-200 dark:bg-slate-800'}">All</button>` + 
                           CATEGORIES.map(c => `<button onclick="filterCat='${c}'; render()" class="px-5 py-2 rounded-full text-xs font-bold ${filterCat === c ? 'bg-indigo-600 text-white' : 'bg-slate-200 dark:bg-slate-800'}">${c}</button>`).join('');

        if (activeTab === 'stories') {
            content.innerHTML = stories.filter(s => !filterCat || s.cats.includes(filterCat)).map(s => `
                <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 p-8 rounded-3xl shadow-sm">
                    <h2 class="text-3xl font-extrabold mb-2 text-slate-900 dark:text-white">${s.title}</h2>
                    <div class="flex gap-2 mb-6">${s.cats.map(c => `<span class="text-[10px] font-bold uppercase tracking-widest text-indigo-600">${c}</span>`).join('')}</div>
                    ${s.blocks.map(b => b.type === 'text' ? `<p class="mb-6 leading-relaxed text-slate-700 dark:text-slate-300 whitespace-pre-line">${b.content}</p>` : `<img src="${b.url}" class="w-full rounded-2xl mb-6 border border-slate-200 dark:border-slate-700">`).join('')}
                </div>
            `).join('');
        } else if (activeTab === 'collections') {
            const series = [...new Set(stories.map(s => s.series).filter(s => s))];
            content.innerHTML = series.map(s => `<div class="p-8 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-3xl font-bold text-lg">${s} <span class="text-indigo-500">(${stories.filter(st => st.series === s).length} parts)</span></div>`).join('');
        } else if (activeTab === 'tags') {
            const allTags = [...new Set(stories.flatMap(s => s.tags).filter(t => t.trim()))];
            content.innerHTML = `<div class="grid grid-cols-2 md:grid-cols-4 gap-4">${allTags.map(t => `<button onclick="setTab('stories', '${t}')" class="p-6 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-2xl font-bold hover:border-indigo-500">#${t.trim()}</button>`).join('')}</div>`;
        } else {
            content.innerHTML = docs.map(d => `<div class="p-8 bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-3xl font-bold text-lg">${d.title}</div>`).join('');
        }
    }
    render();
</script>
</body>
</html>

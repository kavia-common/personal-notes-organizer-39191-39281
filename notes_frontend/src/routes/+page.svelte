<script lang="ts">
  import '../app.css';
  import Sidebar from '$lib/Sidebar.svelte';
  import Editor from '$lib/Editor.svelte';
  import { ui, selectedNote } from '$lib/stores';
  import { listNotes, createNote, updateNote, deleteNoteApi, type Note, type NoteInput } from '$lib/api';

  // local snapshots of stores (Svelte 5: use $state for reactivity)
  let uiState = $state<{
    selectedId: string | null;
    search: string;
    tag: string;
    notes: Note[];
    loading: boolean;
    error: string | null;
  }>({ selectedId: null, search: '', tag: '', notes: [], loading: false, error: null });
  let currentNote = $state<Note | null>(null);

  $effect.root(() => {
    const unsub1 = ui.subscribe((v) => (uiState = v));
    const unsub2 = selectedNote.subscribe((v) => (currentNote = v));
    return () => {
      unsub1();
      unsub2();
    };
  });

  function errorMessage(e: unknown): string {
    if (e instanceof Error) return e.message;
    try {
      return JSON.stringify(e);
    } catch {
      return 'Unknown error';
    }
  }

  // Initial fetch
  async function refresh(): Promise<void> {
    ui.update((s) => ({ ...s, loading: true, error: null }));
    try {
      const notes = await listNotes({ q: uiState.search, tag: uiState.tag });
      ui.update((s) => ({ ...s, notes, loading: false }));
      // If no selection, auto-select first
      const stillExists = uiState.selectedId && notes.some((n) => n.id === uiState.selectedId);
      if (!stillExists) {
        ui.update((s) => ({ ...s, selectedId: notes[0]?.id ?? null }));
      }
    } catch (e) {
      ui.update((s) => ({ ...s, loading: false, error: errorMessage(e) }));
    }
  }

  // Load on mount
  $effect(() => {
    refresh();
  });

  function onSearch(q: string): void {
    ui.update((s) => ({ ...s, search: q }));
    refresh();
  }
  function onTag(t: string): void {
    ui.update((s) => ({ ...s, tag: t }));
    refresh();
  }

  async function onCreate(): Promise<void> {
    // optimistic new note
    const tempId = 'temp-' + Math.random().toString(36).slice(2, 8);
    const draft: Note = {
      id: tempId,
      title: 'Untitled',
      content: '',
      tags: [],
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString()
    };
    ui.update((s) => ({ ...s, notes: [draft, ...s.notes], selectedId: tempId }));

    try {
      const created = await createNote({ title: draft.title, content: draft.content, tags: draft.tags } as NoteInput);
      // replace temp with actual
      ui.update((s) => ({
        ...s,
        notes: s.notes.map((n) => (n.id === tempId ? created : n)),
        selectedId: created.id
      }));
    } catch (e) {
      // rollback
      ui.update((s) => ({
        ...s,
        notes: s.notes.filter((n) => n.id !== tempId),
        selectedId: s.selectedId === tempId ? null : s.selectedId,
        error: errorMessage(e)
      }));
    }
  }

  async function onSave(payload: { title: string; content: string; tags: string[] }): Promise<void> {
    if (!currentNote) return;
    const id = currentNote.id;
    const prev = currentNote;
    // optimistic update
    ui.update((s) => ({
      ...s,
      notes: s.notes.map((n) => (n.id === id ? { ...n, ...payload, updatedAt: new Date().toISOString() } : n))
    }));
    try {
      const updated = await updateNote(id, payload);
      ui.update((s) => ({
        ...s,
        notes: s.notes.map((n) => (n.id === id ? updated : n))
      }));
    } catch (e) {
      // rollback
      ui.update((s) => ({
        ...s,
        notes: s.notes.map((n) => (n.id === id ? prev : n)),
        error: errorMessage(e)
      }));
    }
  }

  async function onDelete(): Promise<void> {
    if (!currentNote) return;
    const id = currentNote.id;
    const prev = currentNote;
    // optimistic remove
    ui.update((s) => ({
      ...s,
      notes: s.notes.filter((n) => n.id !== id),
      selectedId: s.selectedId === id ? null : s.selectedId
    }));
    try {
      await deleteNoteApi(id);
      // if no selected, pick first
      ui.update((s) => (s.selectedId ? s : { ...s, selectedId: s.notes[0]?.id ?? null }));
    } catch (e) {
      // rollback
      ui.update((s) => ({
        ...s,
        notes: [prev, ...s.notes],
        selectedId: id,
        error: errorMessage(e)
      }));
    }
  }

  function onSelect(id: string): void {
    ui.update((s) => ({ ...s, selectedId: id }));
  }
</script>

<svelte:head>
  <title>Personal Notes Organizer</title>
  <meta name="description" content="Create, manage, and organize personal notes." />
</svelte:head>

<div class="app-shell">
  <Sidebar
    {onCreate}
    onSelect={onSelect}
    onSearch={onSearch}
    onTag={onTag}
    notes={uiState.notes}
    loading={uiState.loading}
    error={uiState.error}
    search={uiState.search}
    tag={uiState.tag}
    selectedId={uiState.selectedId}
  />
  <main class="main">
    <div class="spread" style="margin-bottom: 16px;">
      <div>
        <h1 style="margin:0;">Notes</h1>
        <div class="small muted">Create, edit, and organize your thoughts</div>
      </div>
      <div class="row">
        <button class="btn ghost" onclick={refresh}>Refresh</button>
      </div>
    </div>

    <Editor note={currentNote} onSave={onSave} onDelete={onDelete} />
  </main>
</div>

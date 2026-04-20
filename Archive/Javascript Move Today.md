<%*
const folder = `Daily/Journal/${tp.date.now("YYYY")}/${tp.date.now("MM-MMM")}`;
if (!tp.file.exists(folder)) {
    await app.vault.createFolder(folder);
}
await tp.file.move(`${folder}/${tp.file.title}`);
%>
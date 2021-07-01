### Generate Markdown Docs ###
This is to generate proper links for API calls in the table of contents

#### iOS ####
`pandoc API\ Docs\ Android.docx --from docx  --to gfm --table-of-contents --standalone -o API.Android.md`

#### iOS ####
`pandoc API\ Docs\ iOS.docx --from docx  --to gfm --table-of-contents --standalone -o API.iOS.md`
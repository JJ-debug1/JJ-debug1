const {SUDO} = require('../config');
const Heroku = require('heroku-client');
const heroku = new Heroku({token: process.env.HEROKU_API_KEY});
const baseURI = '/apps/' + process.env.HEROKU_APP_NAME;
const {Function} = require("../lib/");
Function({pattern: 'setsudo ?(.*)', fromMe: true, desc: 'set sudo', type: 'user'}, async (m, mm) => {
var newSudo = ( m.reply_message ? m.reply_message.sender : '' || mm).split("@")[0]
if (!newSudo) return await m.send("*reply to a number*",{quoted: m.data});
var setSudo = (SUDO+","+newSudo).replace(/,,/g,",");
setSudo = setSudo.startsWith(",")?setSudo.replace(",",""):setSudo
await m.send('```new sudo numbers are: ```'+setSudo,{quoted: m.data})
await m.send('_It takes 30 seconds to make effect_',{quoted: m.data});
await heroku.patch(baseURI + '/config-vars', {body: { "SUDO": setSudo}}).then(async (app) => {
      });
});
Function({pattern: 'delsudo ?(.*)', fromMe: true, desc: 'delete sudo sudo', type: 'user'}, async (m, mm) => {
var newSudo = ( m.reply_message ? m.reply_message.sender : '' || mm).split("@")[0]
if (!newSudo) return await m.send("*Need reply/mention/number*")
var setSudo = SUDO.replace(newSudo,"").replace(/,,/g,",");
setSudo = setSudo.startsWith(",")?setSudo.replace(",",""):setSudo
await m.send('```NEW SUDO NUMBERS ARE: ```'+setSudo,{quoted: m.data})
await m.send('_IT TAKES 30 SECONDS TO MAKE EFFECT_',{quoted: m.data})
await heroku.patch(baseURI + '/config-vars', {body: { "SUDO": setSudo}}).then(async (app) => {
});
});
Function({pattern: 'getsudo ?(.*)', fromMe: true, desc: 'shows sudo', type: 'user'}, async (m) => {
const vars = await heroku
			.get(baseURI + '/config-vars')
			.catch(async (error) => {
				return await m.send('HEROKU : ' + error.body.message)
			})
		await m.send('```' + `SUDO Numbers are : ${vars.SUDO}` + '```')
	}
)

var express = require('express');
var formidable = require('formidable');
const mongoose = require('mongoose');
var cors = require('cors');
var jwt = require('jsonwebtoken');
require('dotenv').config();
var randomize = require('randomatic');
var bodyParser = require('body-parser');
var jsonParser = bodyParser.json();
var app = express();
var cookieParser = require('cookie-parser')
var i = 0;
var port = process.env.PORT || 4000;
const fs = require('fs');
const Path = require('path');
var fileExtension = require('file-extension');
var favicon = require('serve-favicon')
var path = require('path')
var filesname = [];
var dir = require('node-dir');
const chalk = require('chalk');
const fileNAMES = [];
const fileTYPES = [];
app.use(express.static('./public'));
app.use(favicon(path.join(__dirname, '/public', '/favicon.ico')));
app.use(cookieParser())
app.use(express.static('Semantic-UI-master'));



//DELECTION FUNCTION
const deleteFolderRecursive = function(path) {
    if (fs.existsSync(path)) {
        fs.readdirSync(path).forEach((file, index) => {
            const curPath = Path.join(path, file);
            if (fs.lstatSync(curPath).isDirectory()) { // recurse
                deleteFolderRecursive(curPath);
            } else { // delete file
                fs.unlinkSync(curPath);
            }
        });
        fs.rmdirSync(path);
    }
};


//mongo
mongoose.connect(process.env.DATABASE, { useNewUrlParser: true, useUnifiedTopology: true, useCreateIndex: true }).then(() => {
    console.log(chalk.green.bold("DONE"));
});
const userSchema = new mongoose.Schema({
    usercode: {
        type: String
    },
    uploads: {
        type: String,
        contentType: String
    },
	userCookie:String,
    files: [String],
    filesType: [String]

}, { timestamps: true });
const user = mongoose.model("user", userSchema);
//mongo
app.get("/", (req, res) => {
	 const cok = randomize('Aa', 4);
	console.log(req.cookies); ///PREVIOUS COOKIE
	res.cookie('Name', cok).render("one.ejs");
})
app.get('/send', function(req, res) {
	console.log(chalk.blue.bold(req.cookies.Name)); //NEW COOKIES KEY:Name
    res.render("land.ejs",{msg:""});
});

var usercode = "";


app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.post('/', function(req, res, file) {
    //console.log(req);
	
    const u = new user();
	console.log(chalk.blue.bold(req.cookies.Name)+" / post upload");  
    var form = new formidable.IncomingForm();
	console.log(chalk.blue(JSON.stringify(form)));
    const g = randomize('Aa', 4);
    usercode = g;
    form.parse(req, (error, fields, files)=>{
		console.log(files);
		if(files.size>1048560){
		   res.render("land.ejs",{msg:"Kindly Upload File less than 10MB "});
		   }
		else{
			
			
		}
	});

    fs.mkdir('public/' + req.cookies.Name, err => {
        if (err) {
            console.log(err);
        } else {
            console.log('Directory Created');
        }
    })
    form.on('fileBegin', function(name, file) {
	try{
        file.path = __dirname + '/public/' + req.cookies.Name + "/" + file.name;
        console.log(__dirname);
        console.log(file.path);

        console.log(req.cookies.Name);}
		catch(exce){
			res.send("Error Please try again");
			res.redirect("/");
		}

    });


    form.on('end', () => {
        u.usercode =req.cookies.Name ;
		u.userCookie=req.cookies.Name;
        const filenameArray = [];
        const filetypeArray = [];
        dir.files('public/' + req.cookies.Name, 'file', function(err, content) {
                if (err) throw err;
                content.forEach((files) => {
                    console.log('files : ', files); // get content of files
                    filenameArray[i] = files.slice(12, ); //here we are taking the file name alone
                    u.files.push(files.slice(12, ));
                    console.log('Content Type :', fileExtension(files));
                    u.filesType.push(fileExtension(files));
                    filetypeArray[i] = fileExtension(files);
                    i++;
                    console.log(typeof JSON.stringify(filenameArray));
                    console.log(typeof filetypeArray);

                })
                fileNAMES[0] = JSON.stringify(filenameArray);
                fileTYPES[0] = JSON.stringify(filetypeArray);
                console.log(typeof fileNAMES[0] + "==========" + typeof fileTYPES[0]);
                console.log(fileNAMES[0] + "==========" + fileTYPES[0]);

                u.save((err, ok) => {
                    if (err) {
                        console.log("Error in saving")
                    } else {
                        console.log(ok);
						res.render("donePage.ejs");
                       // res.render("viewpage.ejs", { done: ok, allfiles: content,usercode:usercode });
                    }
                });
            },
            function(err, files) {
                if (err) throw err;
                console.log('finished reading files:'); // get filepath 
            });


    });




});
//SEND CODE BLOCK WITH COOKIE
app.post("/getUserCode",(req,res)=>{
		console.log(chalk.blue.bold(req.cookies.Name)+" GET USER-CODE WITH COOKIE BLOCK");
		  
			user.findOne({userCookie:req.cookies.Name},(err,getAll)=>{
				if(err){
				   console.log(err);
				   }
				else{
					console.log(getAll);
					console.log(getAll.usercode);

					res.render("codePage.ejs",{usercode:getAll.usercode,allgetuser:getAll,ffname:getAll.files});
				}
			})
});
//RECEIVE BLOCK

app.get("/get", (req, res) => {
    res.render("get.ejs",{msg:""});
});

app.post("/downloadfile", (req, res) => {
	user.find({usercode:req.body.entercode},(err,gh)=>{
		if(err){
		   console.log(err);
		   }
		else{
			console.log(chalk.blue(gh));
		}
	});
	  
	
	try{
    const fileList = "";
    console.log(req.body.entercode + " User need hes/her files");
	
    user.findOne({ usercode: req.body.entercode }, (err, fi) => {
        if (err) {
			console.log(err);
        } else {
            console.log(fi+" |||");
			if(fi===null){
				console.log("There is not file with this code");///  HERE WE SOLVED THE PROBLEM OF UPLOAD INVALID CODE POROBLEM
			
				res.render("get.ejs",{msg:"please enter the valid code!!"});
			}
			else if(fi!=null){
            console.log(fi.files);
            console.log(fi.filesType);
            console.log(fi._id);
            console.log(fi.createdAt);
            console.log(fi.usercode);
            res.render("receive.ejs", { Rfiles: fi, filesName: fi.files, userCode: fi.usercode });
		}
		}
    })}
	
	catch(e){
		res.redirect("/");
	}
});
app.post("/end", (req, res) => {
    console.log(req.body.Code);
    console.log("END SESSION " + req.body.Code);
    user.findOneAndDelete({ usercode: req.body.Code }, (err, deleUsercode) => {
        if (err) {
			res.redirect("/");
            console.log("ERROR END SESSION");
        } else {
            console.log(deleUsercode);
        }
    });
    deleteFolderRecursive('./public/' + req.body.Code);
    res.redirect("/");
});


app.listen(port, () => {
    console.log("Yes connected");
});
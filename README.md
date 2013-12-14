Chatbot
=======
(function() {
  var Command, RoomHelper, User, addCommand, afkCheck, afksCommand, allAfksCommand, announceCurate, antispam, apiHooks, avgVoteRatioCommand, badQualityCommand, beggar, chatCommandDispatcher, chatUniversals, cmds, commandsCommand, cookieCommand, data, dieCommand, disconnectLookupCommand, downloadCommand, forceSkipCommand, handleNewSong, handleUserJoin, handleUserLeave, handleVote, hook, initEnvironment, initHooks, initialize, lockCommand, lockskipCommand, msToStr, newSongsCommand, overplayedCommand, popCommand, populateUserData, pupOnline, pushCommand, reloadCommand, resetAfkCommand, roomHelpCommand, rulesCommand, settings, skipCommand, staffCommand, sourceCommand, statusCommand, swapCommand, themeCommand, undoHooks, unhook, unhookCommand, unlockCommand, updateVotes, uservoiceCommand, voteRatioCommand, whyMehCommand, whyWootCommand, wootCommand, _ref, _ref1, _ref10, _ref11, _ref12, _ref13, _ref14, _ref15, _ref16, _ref17, _ref18, _ref19, _ref2, _ref20, _ref21, _ref22, _ref23, _ref24, _ref25, _ref26, _ref27, _ref28, _ref29, _ref3, _ref30, _ref32, ref33, _ref34, ref35, _ref4, _ref5, _ref6, _ref7, _ref8, _ref9,
    __bind = function(fn, me){ return function(){ return fn.apply(me, arguments); }; },
    __indexOf = [].indexOf || function(item) { for (var i = 0, l = this.length; i < l; i++) { if (i in this && this[i] === item) return i; } return -1; },
    __hasProp = {}.hasOwnProperty,
    __extends = function(child, parent) { for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; } function ctor() { this.constructor = child; } ctor.prototype = parent.prototype; child.prototype = new ctor(); child.__super__ = parent.prototype; return child; };

  settings = (function() {
    function settings() {
      this.implode = __bind(this.implode, this);
      this.intervalMessages = __bind(this.intervalMessages, this);
      this.startAfkInterval = __bind(this.startAfkInterval, this);
      this.setInternalWaitlist = __bind(this.setInternalWaitlist, this);
      this.userJoin = __bind(this.userJoin, this);
      this.getRoomUrlPath = __bind(this.getRoomUrlPath, this);
      this.startup = __bind(this.startup, this);
    }

    settings.prototype.currentsong = {};

    settings.prototype.users = {};

    settings.prototype.djs = [];

    settings.prototype.mods = [];

    settings.prototype.host = [];

    settings.prototype.hasWarned = false;

    settings.prototype.currentwoots = 0;

    settings.prototype.currentmehs = 0;

    settings.prototype.currentcurates = 0;

    settings.prototype.roomUrlPath = null;

    settings.prototype.internalWaitlist = [];

    settings.prototype.userDisconnectLog = [];

    settings.prototype.voteLog = {};

    settings.prototype.seshOn = false;

    settings.prototype.forceSkip = false;

    settings.prototype.seshMembers = [];

    settings.prototype.launchTime = null;

    settings.prototype.totalVotingData = {
      woots: 0,
      mehs: 0,
      curates: 0
    };

    settings.prototype.pupScriptUrl = 'https://github.com/igorce9/Chatbot/master/README.md';

    settings.prototype.afkTime = 15 * 60 * 1000;

    settings.prototype.songIntervalMessages = [
      {
        interval: 7,
        offset: 0,
        msg: "/em: Mantenha-se sempre votando, se não será removido da lista de espera e da cabine de DJ."
      },{
        interval: 5,
        offset: 0,
        msg: "/em: Make sure that you're voting or you will be removed from the wait list and from the DJ Booth."
      },{
        interval: 9,
        offset: 0,
        msg: "/me Don't Ask for fans :V:"
      }
    ];

    settings.prototype.songCount = 0;

    settings.prototype.startup = function() {
      this.launchTime = new Date();
      return this.roomUrlPath = this.getRoomUrlPath();
    };

    settings.prototype.getRoomUrlPath = function() {
      return window.location.pathname.replace(/\//g, '');
    };

    settings.prototype.newSong = function() {
      this.totalVotingData.woots += this.currentwoots;
      this.totalVotingData.mehs += this.currentmehs;
      this.totalVotingData.curates += this.currentcurates;
      this.setInternalWaitlist();
      this.currentsong = API.getMedia();
      if (this.currentsong !== null) {
        return this.currentsong;
      } else {
        return false;
      }
    };

    settings.prototype.userJoin = function(u) {
      var userIds, _ref;
      userIds = Object.keys(this.users);
      if (_ref = u.id, __indexOf.call(userIds, _ref) >= 0) {
        return this.users[u.id].inRoom(true);
      } else {
        this.users[u.id] = new User(u);
        return this.voteLog[u.id] = {};
      }
    };

    settings.prototype.setInternalWaitlist = function() {
      var boothWaitlist, fullWaitList, lineWaitList;
      boothWaitlist = API.getDJs().slice(1);
      lineWaitList = API.getWaitList();
      fullWaitList = boothWaitlist.concat(lineWaitList);
      return this.internalWaitlist = fullWaitList;
    };

    settings.prototype.activity = function(obj) {
      if (obj.type === 'message') {
        return this.users[obj.fromID].updateActivity();
      }
    };

    settings.prototype.startAfkInterval = function() {
      return this.afkInterval = setInterval(afkCheck, 2000);
    };

    settings.prototype.intervalMessages = function() {
      var msg, _i, _len, _ref, _results;
      this.songCount++;
      _ref = this.songIntervalMessages;
      _results = [];
      for (_i = 0, _len = _ref.length; _i < _len; _i++) {
        msg = _ref[_i];
        if (((this.songCount + msg['offset']) % msg['interval']) === 0) {
          _results.push(API.sendChat(msg['msg']));
        } else {
          _results.push(void 0);
        }
      }
      return _results;
    };

    settings.prototype.implode = function() {
      var item, val;
      for (item in this) {
        val = this[item];
        if (typeof this[item] === 'object') {
          delete this[item];
        }
      }
      return clearInterval(this.afkInterval);
    };

    settings.prototype.lockBooth = function(callback) {
      if (callback == null) {
        callback = null;
      }
      return $.ajax({
        url: "http://plug.dj/_/gateway/room.update_options",
        type: 'POST',
        data: JSON.stringify({
          service: "room.update_options",
          body: [
            this.roomUrlPath, {
              "boothLocked": true,
              "waitListEnabled": true,
              "maxPlays": 1,
              "maxDJs": 5
            }
          ]
        }),
        async: this.async,
        dataType: 'json',
        contentType: 'application/json'
      }).done(function() {
        if (callback != null) {
          return callback();
        }
      });
    };

    settings.prototype.unlockBooth = function(callback) {
      if (callback == null) {
        callback = null;
      }
      return $.ajax({
        url: "http://plug.dj/_/gateway/room.update_options",
        type: 'POST',
        data: JSON.stringify({
          service: "room.update_options",
          body: [
            this.roomUrlPath, {
              "boothLocked": false,
              "waitListEnabled": true,
              "maxPlays": 1,
              "maxDJs": 5
            }
          ]
        }),
        async: this.async,
        dataType: 'json',
        contentType: 'application/json'
      }).done(function() {
        if (callback != null) {
          return callback();
        }
      });
    };

    return settings;

  })();

  data = new settings();

  User = (function() {
    User.prototype.afkWarningCount = 0;

    User.prototype.lastWarning = null;

    User.prototype["protected"] = false;

    User.prototype.isInRoom = true;

    function User(user) {
      this.user = user;
      this.updateVote = __bind(this.updateVote, this);
      this.inRoom = __bind(this.inRoom, this);
      this.notDj = __bind(this.notDj, this);
      this.warn = __bind(this.warn, this);
      this.getIsDj = __bind(this.getIsDj, this);
      this.getWarningCount = __bind(this.getWarningCount, this);
      this.getUser = __bind(this.getUser, this);
      this.getLastWarning = __bind(this.getLastWarning, this);
      this.getLastActivity = __bind(this.getLastActivity, this);
      this.updateActivity = __bind(this.updateActivity, this);
      this.init = __bind(this.init, this);
      this.init();
    }

    User.prototype.init = function() {
      return this.lastActivity = new Date();
    };

    User.prototype.updateActivity = function() {
      this.lastActivity = new Date();
      this.afkWarningCount = 0;
      return this.lastWarning = null;
    };

    User.prototype.getLastActivity = function() {
      return this.lastActivity;
    };

    User.prototype.getLastWarning = function() {
      if (this.lastWarning === null) {
        return false;
      } else {
        return this.lastWarning;
      }
    };

    User.prototype.getUser = function() {
      return this.user;
    };

    User.prototype.getWarningCount = function() {
      return this.afkWarningCount;
    };

    User.prototype.getIsDj = function() {
      var DJs, dj, _i, _len;
      DJs = API.getDJs();
      for (_i = 0, _len = DJs.length; _i < _len; _i++) {
        dj = DJs[_i];
        if (this.user.id === dj.id) {
          return true;
        }
      }
      return false;
    };

    User.prototype.warn = function() {
      this.afkWarningCount++;
      return this.lastWarning = new Date();
    };

    User.prototype.notDj = function() {
      this.afkWarningCount = 0;
      return this.lastWarning = null;
    };

    User.prototype.inRoom = function(online) {
      return this.isInRoom = online;
    };

    User.prototype.updateVote = function(v) {
      if (this.isInRoom) {
        return data.voteLog[this.user.id][data.currentsong.id] = v;
      }
    };

    return User;

  })();

  RoomHelper = (function() {
    function RoomHelper() {}

    RoomHelper.prototype.lookupUser = function(username) {
      var id, u, _ref;
      _ref = data.users;
      for (id in _ref) {
        u = _ref[id];
        if (u.getUser().username === username) {
          return u.getUser();
        }
      }
      return false;
    };

    RoomHelper.prototype.userVoteRatio = function(user) {
      var songId, songVotes, vote, votes;
      songVotes = data.voteLog[user.id];
      votes = {
        'woot': 0,
        'meh': 0
      };
      for (songId in songVotes) {
        vote = songVotes[songId];
        if (vote === 1) {
          votes['woot']++;
        } else if (vote === -1) {
          votes['meh']++;
        }
      }
      votes['positiveRatio'] = (votes['woot'] / (votes['woot'] + votes['meh'])).toFixed(2);
      return votes;
    };

    return RoomHelper;

  })();

  pupOnline = function() {
    return API.sendChat("/em: ChatBot ON Corre Negada!");
  };

  populateUserData = function() {
    var u, users, _i, _len;
    users = API.getUsers();
    for (_i = 0, _len = users.length; _i < _len; _i++) {
      u = users[_i];
      data.users[u.id] = new User(u);
      data.voteLog[u.id] = {};
    }
  };

  initEnvironment = function() {
    document.getElementById("button-vote-positive").click();
    return document.getElementById("button-sound").click();
  };

  initialize = function() {
    pupOnline();
    populateUserData();
    initEnvironment();
    initHooks();
    data.startup();
    data.newSong();
    return data.startAfkInterval();
  };

  afkCheck = function() {
    var DJs, id, lastActivity, lastWarned, now, oneMinute, secsLastActive, timeSinceLastActivity, timeSinceLastWarning, twoMinutes, user, warnMsg, _ref, _results;
    _ref = data.users;
    _results = [];
    for (id in _ref) {
      user = _ref[id];
      now = new Date();
      lastActivity = user.getLastActivity();
      timeSinceLastActivity = now.getTime() - lastActivity.getTime();
      if (timeSinceLastActivity > data.afkTime) {
        if (user.getIsDj()) {
          secsLastActive = timeSinceLastActivity / 1000;
          if (user.getWarningCount() === 0) {
            user.warn();
            _results.push(API.sendChat("@" + user.getUser().username + ", Eu não vi você votar durante 30 minutos.  Vote nos próximos 2 minutos, se não será removido."));
          } else if (user.getWarningCount() === 1) {
            lastWarned = user.getLastWarning();
            timeSinceLastWarning = now.getTime() - lastWarned.getTime();
            twoMinutes = 1 * 60 * 1000;
            if (timeSinceLastWarning > twoMinutes) {
              user.warn();
              warnMsg = "@" + user.getUser().username;
              warnMsg += ", Eu não vi você votar durante 32 minutos. Vote no próximo minuto ou eu o removerei, este é seu ultimo aviso.";
              _results.push(API.sendChat(warnMsg));
            } else {
              _results.push(void 0);
            }
          } else if (user.getWarningCount() === 2) {
            lastWarned = user.getLastWarning();
            timeSinceLastWarning = now.getTime() - lastWarned.getTime();
            oneMinute = 1 * 60 * 1000;
            if (timeSinceLastWarning > oneMinute) {
              DJs = API.getDJs();
              if (DJs.length > 0 && DJs[0].id !== user.getUser().id) {
                API.sendChat("@" + user.getUser().username + ", Você teve 2 avisos. Por favor, mantenha-se sempre votando.");
                API.moderateRemoveDJ(id);
                _results.push(user.warn());
              } else {
                _results.push(void 0);
              }
            } else {
              _results.push(void 0);
            }
          } else {
            _results.push(void 0);
          }
        } else {
          _results.push(user.notDj());
        }
      } else {
        _results.push(void 0);
      }
    }
    return _results;
  };

  msToStr = function(msTime) {
    var ms, msg, timeAway;
    msg = '';
    timeAway = {
      'days': 0,
      'hours': 0,
      'minutes': 0,
      'seconds': 0
    };
    ms = {
      'day': 24 * 60 * 60 * 1000,
      'hour': 60 * 60 * 1000,
      'minute': 60 * 1000,
      'second': 1000
    };
    if (msTime > ms['day']) {
      timeAway['days'] = Math.floor(msTime / ms['day']);
      msTime = msTime % ms['day'];
    }
    if (msTime > ms['hour']) {
      timeAway['hours'] = Math.floor(msTime / ms['hour']);
      msTime = msTime % ms['hour'];
    }
    if (msTime > ms['minute']) {
      timeAway['minutes'] = Math.floor(msTime / ms['minute']);
      msTime = msTime % ms['minute'];
    }
    if (msTime > ms['second']) {
      timeAway['seconds'] = Math.floor(msTime / ms['second']);
    }
    if (timeAway['days'] !== 0) {
      msg += timeAway['days'].toString() + 'd';
    }
    if (timeAway['hours'] !== 0) {
      msg += timeAway['hours'].toString() + 'h';
    }
    if (timeAway['minutes'] !== 0) {
      msg += timeAway['minutes'].toString() + 'm';
    }
    if (timeAway['seconds'] !== 0) {
      msg += timeAway['seconds'].toString() + 's';
    }
    if (msg !== '') {
      return msg;
    } else {
      return false;
    }
  };

  Command = (function() {
    function Command(msgData) {
      this.msgData = msgData;
      this.init();
    }

    Command.prototype.init = function() {
      this.parseType = null;
      this.command = null;
      return this.rankPrivelege = null;
    };

    Command.prototype.functionality = function(data) {};

    Command.prototype.hasPrivelege = function() {
      var user;
      user = data.users[this.msgData.fromID].getUser();
      switch (this.rankPrivelege) {
        case 'host':
          return user.permission === 5;
        case 'cohost':
          return user.permission >= 4;
        case 'mod':
          return user.permission >= 3;
        case 'manager':
          return user.permission >= 3;
        case 'bouncer':
          return user.permission >= 2;
        case 'featured':
          return user.permission >= 1;
        default:
          return true;
      }
    };

    Command.prototype.commandMatch = function() {
      var command, msg, _i, _len, _ref;
      msg = this.msgData.message;
      if (typeof this.command === 'string') {
        if (this.parseType === 'exact') {
          if (msg === this.command) {
            return true;
          } else {
            return false;
          }
        } else if (this.parseType === 'startsWith') {
          if (msg.substr(0, this.command.length) === this.command) {
            return true;
          } else {
            return false;
          }
        } else if (this.parseType === 'contains') {
          if (msg.indexOf(this.command) !== -1) {
            return true;
          } else {
            return false;
          }
        }
      } else if (typeof this.command === 'object') {
        _ref = this.command;
        for (_i = 0, _len = _ref.length; _i < _len; _i++) {
          command = _ref[_i];
          if (this.parseType === 'exact') {
            if (msg === command) {
              return true;
            }
          } else if (this.parseType === 'startsWith') {
            if (msg.substr(0, command.length) === command) {
              return true;
            }
          } else if (this.parseType === 'contains') {
            if (msg.indexOf(command) !== -1) {
              return true;
            }
          }
        }
        return false;
      }
    };

    Command.prototype.evalMsg = function() {
      if (this.commandMatch() && this.hasPrivelege()) {
        this.functionality();
        return true;
      } else {
        return false;
      }
    };

    return Command;

  })();

  cookieCommand = (function(_super) {
    __extends(cookieCommand, _super);

    function cookieCommand() {
      _ref = cookieCommand.__super__.constructor.apply(this, arguments);
      return _ref;
    }

    cookieCommand.prototype.init = function() {
      this.command = 'cookie';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'featured';
    };

    cookieCommand.prototype.getCookie = function() {
      var c, cookies;
      cookies = ["muita aveia", "um cafesinho", "um toddynho", "um copo de coca-cola", "uma dose de Ipioca", "limões", "um danoninho", "um cafe extra forte", "maçãs", "bananas", "um brinquedinho", "amendoins", "muita aveia"];
      c = Math.floor(Math.random() * cookies.length);
      return cookies[c];
    };

    cookieCommand.prototype.functionality = function() {
      var msg, r, user;
      msg = this.msgData.message;
      r = new RoomHelper();
      if (msg.substring(7, 8) === "@") {
        user = r.lookupUser(msg.substr(8));
        if (user === false) {
          API.sendChat("/em: não encontrei ' " + msg.substr(8) +  "' na sala. Então, sobra mais pra mim! d(>_<)b ");
          return false;
        } else {
          return API.sendChat("/em: @" + user.username + ", @" + this.msgData.from + " o recompensou com " + this.getCookie() + ".  Aproveite!");
        }
      }
    };

    return cookieCommand;

  })(Command);

  newSongsCommand = (function(_super) {
    __extends(newSongsCommand, _super);

    function newSongsCommand() {
      _ref1 = newSongsCommand.__super__.constructor.apply(this, arguments);
      return _ref1;
    }

    newSongsCommand.prototype.init = function() {
      this.command = '!musicanova';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'featured';
    };

    newSongsCommand.prototype.functionality = function() {
      var arts, cMedia, chans, chooseRandom, mChans, msg, selections, u, _ref2;
      mChans = this.memberChannels.slice(0);
      chans = this.channels.slice(0);
      arts = this.artists.slice(0);
      chooseRandom = function(list) {
        var l, r;
        l = list.length;
        r = Math.floor(Math.random() * l);
        return list.splice(r, 1);
      };
      selections = {
        channels: [],
        artist: ''
      };
      u = data.users[this.msgData.fromID].getUser().username;
      if (u.indexOf("MistaDubstep") !== -1) {
        selections['channels'].push('MistaDubstep');
      } else if (u.indexOf("Underground Promotions") !== -1) {
        selections['channels'].push('UndergroundDubstep');
      } else {
        selections['channels'].push(chooseRandom(mChans));
      }
      selections['channels'].push(chooseRandom(chans));
      selections['channels'].push(chooseRandom(chans));
      cMedia = API.getMedia();
      if ((cMedia != null) && (_ref2 = cMedia.author, __indexOf.call(arts, _ref2) >= 0)) {
        selections['artist'] = cMedia.author;
      } else {
        selections['artist'] = chooseRandom(arts);
      }
     msg = "Todo mundo já ouviu aquela musica do " + selections['artist'] + " obtenha novas musicas em http://youtube.com/" + selections['channels'][0] + " http://youtube.com/" + selections['channels'][1] + " ou http://youtube.com/" + selections['channels'][2];
      return API.sendChat(msg);
    };

    newSongsCommand.prototype.memberChannels = ["JitterStep", "MistaDubstep", "DubStationPromotions", "UndergroundDubstep",, "DarkstepWarrior", "BombshockDubstep", "Sharestep"];

    newSongsCommand.prototype.channels = ["UltraRecords", "kontor", "SpinninRec", "PandoraMuslc", "ITacMusic", "Liquicity", "FunkyyPanda", "BassRape", "WobbleCraftDubz", "UKFdubstep", "DropThatBassline", "VitalDubstep", "AirwaveDubstepTV", "EpicNetworkMusic", "NoOffenseDubstep", "InspectorDubplate", "ReptileDubstep", "MrMoMDubstep", "FrixionNetwork", "IcyDubstep", "DubstepWeed", "VhileMusic", "LessThan3Dubstep", "PleaseMindTheDUBstep", "ClownDubstep", "TheULTRADUBSTEP", "DuBM0nkeyz", "DubNationUK", "TehDubstepChannel", "BassDropMedia", "USdubstep", "UNITEDubstep"];

    newSongsCommand.prototype.artists = ["Skrillex", "Doctor P", "Excision", "Flux Pavilion", "Knife Party", "Krewella", "Rusko", "Bassnectar", "Nero", "Deadmau5", "Borgore", "Zomboy"];

    return newSongsCommand;

  })(Command);

  whyWootCommand = (function(_super) {
    __extends(whyWootCommand, _super);

    function whyWootCommand() {
      _ref2 = whyWootCommand.__super__.constructor.apply(this, arguments);
      return _ref2;
    }

    whyWootCommand.prototype.init = function() {
      this.command = '!brutos';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'featured';
    };

    whyWootCommand.prototype.functionality = function() {
      var msg, nameIndex;
      msg = "lindão vlw flw!";;
      if ((nameIndex = this.msgData.message.indexOf('@')) !== -1) {
        return API.sendChat(this.msgData.message.substr(nameIndex) + ', ' + msg);
      } else {
        return API.sendChat(msg);
      }
    };

    return whyWootCommand;

  })(Command);

  themeCommand = (function(_super) {
    __extends(themeCommand, _super);

    function themeCommand() {
      _ref3 = themeCommand.__super__.constructor.apply(this, arguments);
      return _ref3;
    }

    themeCommand.prototype.init = function() {
      this.command = '!temas';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'featured';
    };

    themeCommand.prototype.functionality = function() {
      var msg1, msg2;
      msg1 = "Todo tipo de musica eletronica de boa qualidade é permitida aqui. Incluindo Electro, Dubstep, Techno, Trap, EDM, ";
      msg1 += "Hardstyle, House e Trance. ";
      msg2 = "All kinds of electronic song of good quality is allowed here. Including Electro, Dubstep, Techno, Trap, EDM, ";
      msg2 += "Hardstyle, House and Trance. ";	  
      API.sendChat(msg1);
	  return setTimeout((function() {
        return API.sendChat(msg2);
      }), 750);
	  
    };

    return themeCommand;

  })(Command);

  rulesCommand = (function(_super) {
    __extends(rulesCommand, _super);

    function rulesCommand() {
      _ref4 = rulesCommand.__super__.constructor.apply(this, arguments);
      return _ref4;
    }

    rulesCommand.prototype.init = function() {
      this.command = '!regras';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'featured';
    };

    rulesCommand.prototype.functionality = function() {
      var msg1, msg2;
       msg1 = "1) Tempo máximo 6 min e 30 seg. ";
      msg1 += "2) Não escrever em /me ou /em.  ";
      msg1 += "3) Respeitar os moderadores da sala. ";
      msg1 += "4) Sem flood no chat. ";
      msg1 += "5) Não fique pedindo cargos. ";
      msg1 += "6) Proibido pedir fans ou usar fanbot! ";
      msg2 += "1) Maximum time 6 minutes and 30 seconds. ";
      msg2 += "2) Can't write in /me or /em. ";
      msg2 += "3) Respect the moderators of the room. ";
      msg2 += "4) Without flood in the chat. ";
      msg2 += "5) Can't ask positions. ";
      msg2 += "6  No asf for fans! ";
      API.sendChat(msg1);
      return setTimeout((function() {
        return API.sendChat(msg2);
      }), 750);
    };

    return rulesCommand;

  })(Command);

  roomHelpCommand = (function(_super) {
    __extends(roomHelpCommand, _super);

    function roomHelpCommand() {
      _ref5 = roomHelpCommand.__super__.constructor.apply(this, arguments);
      return _ref5;
    }

    roomHelpCommand.prototype.init = function() {
      this.command = '!ajuda';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'featured';
    };

    roomHelpCommand.prototype.functionality = function() {
      var msg1, msg2;
      msg1 = "Bem vindo a sala Electro House Br! Ser DJ: Crie uma lista de reprodução e coloque Musica do Youtube ou soundcloud. ";
      msg1 += "Se é novo va embaixo do chat e clique onde tera um nome e depois mude o nome. ";
      msg1 += "Para Ganhar Pontos clique em woot. ";
      msg2 = "Divirta-se! Qualquer outra duvida chame um adm da sala ";
      API.sendChat(msg1);
      return setTimeout((function() {
        return API.sendChat(msg2);
      }), 750);
    };

    return roomHelpCommand;

  })(Command);

  sourceCommand = (function(_super) {
    __extends(sourceCommand, _super);

    function sourceCommand() {
      _ref6 = sourceCommand.__super__.constructor.apply(this, arguments);
      return _ref6;
    }

    sourceCommand.prototype.init = function() {
      this.command = ['!autor', '!author', '!autor'];
      this.parseType = 'exact';
      return this.rankPrivelege = 'featured';
    };

    sourceCommand.prototype.functionality = function() {
      var msg;
      msg = ' BOT editado por igorce9 para sala Electro House BR ';
      return API.sendChat(msg);
    };

    return sourceCommand;

  })(Command);

  wootCommand = (function(_super) {
    __extends(wootCommand, _super);

    function wootCommand() {
      _ref7 = wootCommand.__super__.constructor.apply(this, arguments);
      return _ref7;
    }

    wootCommand.prototype.init = function() {
      this.command = '!autowoot';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'featured';
    };

    wootCommand.prototype.functionality = function() {
      var msg, nameIndex;
      msg = "Use o auto woot para evitar remoção da cabine de DJ por não estar votando .http://derpthebass.github.io/BassPlugLite/ ";
      if ((nameIndex = this.msgData.message.indexOf('@')) !== -1) {
        return API.sendChat(this.msgData.message.substr(nameIndex) + ', ' + msg);
      } else {
        return API.sendChat(msg);
      }
    };

    return wootCommand;

  })(Command);

  badQualityCommand = (function(_super) {
    __extends(badQualityCommand, _super);

    function badQualityCommand() {
      _ref8 = badQualityCommand.__super__.constructor.apply(this, arguments);
      return _ref8;
    }

    badQualityCommand.prototype.init = function() {
      this.command = '.ruim';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    badQualityCommand.prototype.functionality = function() {
      var msg;
      msg = "Não gostei da sua musica, tente melhorar da proxima vez";
      return API.sendChat(msg);
    };

    return badQualityCommand;

  })(Command);

  downloadCommand = (function(_super) {
    __extends(downloadCommand, _super);

    function downloadCommand() {
      _ref9 = downloadCommand.__super__.constructor.apply(this, arguments);
      return _ref9;
    }

    downloadCommand.prototype.init = function() {
      this.command = '!fb';
      this.parseType = 'exact';
      return this.rankPrivelege = 'featured';
    };

    downloadCommand.prototype.functionality = function() {
      var msg;
      msg = "  Participe do grupo e curta nossa pagina ";
      msg += "https://www.facebook.com/pages/Electro-House-BR/785188514831094 ";
      msg += "https://www.facebook.com/groups/1378744439017099/ ";
      return API.sendChat(msg);
    };

    return downloadCommand;

  })(Command);

  afksCommand = (function(_super) {
    __extends(afksCommand, _super);

    function afksCommand() {
      _ref10 = afksCommand.__super__.constructor.apply(this, arguments);
      return _ref10;
    }

    afksCommand.prototype.init = function() {
      this.command = '!afks';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    afksCommand.prototype.functionality = function() {
      var dj, djAfk, djs, msg, now, _i, _len;
      msg = '';
      djs = API.getDJs();
      for (_i = 0, _len = djs.length; _i < _len; _i++) {
        dj = djs[_i];
        now = new Date();
        djAfk = now.getTime() - data.users[dj.id].getLastActivity().getTime();
        if (djAfk > (4 * 60 * 1000)) {
          if (msToStr(djAfk) !== false) {
            msg += dj.username + ' - ' + msToStr(djAfk);
            msg += '. ';
          }
        }
      }
      if (msg === '') {
        return API.sendChat("Ninguém está AFK");
      } else {
        return API.sendChat('AFKs: ' + msg);
      }
    };

    return afksCommand;

  })(Command);

  allAfksCommand = (function(_super) {
    __extends(allAfksCommand, _super);

    function allAfksCommand() {
      _ref11 = allAfksCommand.__super__.constructor.apply(this, arguments);
      return _ref11;
    }

    allAfksCommand.prototype.init = function() {
      this.command = '!allafks';
      this.parseType = 'exact';
      return this.rankPrivelege = 'manager';
    };

    allAfksCommand.prototype.functionality = function() {
      var msg, now, u, uAfk, usrs, _i, _len;
      msg = '';
      usrs = API.getUsers();
      for (_i = 0, _len = usrs.length; _i < _len; _i++) {
        u = usrs[_i];
        now = new Date();
        uAfk = now.getTime() - data.users[u.id].getLastActivity().getTime();
        if (uAfk > (30 * 60 * 1000)) {
          if (msToStr(uAfk) !== false) {
            msg += u.username + ' - ' + msToStr(uAfk);
            msg += '. ';
          }
        }
      }
      if (msg === '') {
        return API.sendChat("Ninguém está AFK");
      } else {
        return API.sendChat('AFKs: ' + msg);
      }
    };

    return allAfksCommand;

  })(Command);

  statusCommand = (function(_super) {
    __extends(statusCommand, _super);

    function statusCommand() {
      _ref12 = statusCommand.__super__.constructor.apply(this, arguments);
      return _ref12;
    }

    statusCommand.prototype.init = function() {
      this.command = '!status';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    statusCommand.prototype.functionality = function() {
      var day, hour, launch, lt, meridian, min, month, msg, t;
      lt = data.launchTime;
      month = lt.getMonth() + 1;
      day = lt.getDate();
      hour = lt.getHours();
      meridian = hour % 12 === hour ? 'AM' : 'PM';
      min = lt.getMinutes();
      min = min < 10 ? '0' + min : min;
      t = data.totalVotingData;
      t['songs'] = data.songCount;
      launch = 'Iniciado em ' + day + '/' + month + ' ' + hour + ':' + min + ' ' + meridian + '. ';
      msg = launch;
      return API.sendChat(msg);
    };

    return statusCommand;

  })(Command);

  unhookCommand = (function(_super) {
    __extends(unhookCommand, _super);

    function unhookCommand() {
      _ref13 = unhookCommand.__super__.constructor.apply(this, arguments);
      return _ref13;
    }

    unhookCommand.prototype.init = function() {
      this.command = '!unhook events all()';
      this.parseType = 'exact';
      return this.rankPrivelege = 'host';
    };

    unhookCommand.prototype.functionality = function() {
      API.sendChat('Unhooking all events...');
      return undoHooks();
    };

    return unhookCommand;

  })(Command);

  dieCommand = (function(_super) {
    __extends(dieCommand, _super);

    function dieCommand() {
      _ref14 = dieCommand.__super__.constructor.apply(this, arguments);
      return _ref14;
    }

    dieCommand.prototype.init = function() {
      this.command = '!desligar';
      this.parseType = 'exact';
      return this.rankPrivelege = 'cohost';
    };

    dieCommand.prototype.functionality = function() {
      API.sendChat('Como tu é burro, cara.');
      undoHooks();
      API.sendChat('Por que fez isso? Que loucura.');
      data.implode();
      return API.sendChat('Agora o chat bot está desligado');
    };

    return dieCommand;

  })(Command);

  reloadCommand = (function(_super) {
    __extends(reloadCommand, _super);

    function reloadCommand() {
      _ref15 = reloadCommand.__super__.constructor.apply(this, arguments);
      return _ref15;
    }

    reloadCommand.prototype.init = function() {
      this.command = '!reload';
      this.parseType = 'exact';
      return this.rankPrivelege = 'cohost';
    };

    reloadCommand.prototype.functionality = function() {
      var pupSrc;
      API.sendChat('Reiniciado!');
      undoHooks();
      pupSrc = data.pupScriptUrl;
      data.implode();
      return $.getScript(pupSrc);
    };

    return reloadCommand;

  })(Command);

  lockCommand = (function(_super) {
    __extends(lockCommand, _super);

    function lockCommand() {
      _ref16 = lockCommand.__super__.constructor.apply(this, arguments);
      return _ref16;
    }

    lockCommand.prototype.init = function() {
      this.command = '!trava';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    lockCommand.prototype.functionality = function() {
      return data.lockBooth();
    };

    return lockCommand;

  })(Command);

  unlockCommand = (function(_super) {
    __extends(unlockCommand, _super);

    function unlockCommand() {
      _ref17 = unlockCommand.__super__.constructor.apply(this, arguments);
      return _ref17;
    }

    unlockCommand.prototype.init = function() {
      this.command = '!destrava';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    unlockCommand.prototype.functionality = function() {
      return data.unlockBooth();
    };

    return unlockCommand;
	
  })(Command);

  removeCommand = (function(_super) {
    __extends(removeCommand, _super);

    function removeCommand() {
      _ref33 = removeCommand.__super__.constructor.apply(this, arguments);
      return _ref33;
    }

    removeCommand.prototype.init = function() {
      this.command = '!remove';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'bouncer';
    };

    removeCommand.prototype.functionality = function() {
      var djs, popDj;

      djs = API.getDJs();
      popDj = djs[djs.length - 1];
      return API.moderateRemoveDJ(popDj.id);
    };

    return removeCommand;
	 
	 })(Command);

  staffCommand = (function(_super) {
    __extends(staffCommand, _super);

    function staffCommand() {
      _ref32 = staffCommand.__super__.constructor.apply(this, arguments);
      return _ref32;
    }

    staffCommand.prototype.init = function() {
      this.command = '!staff';
      this.parseType = 'exact';
      this.rankPrivelege = 'user';
      return window.lastActiveStaffTime;
    };
	
    staffCommand.prototype.staff = function() {
      var now, staff, staffAfk, stringstaff, user, _i, _len;

      staff = API.getStaff();
      now = new Date();
      stringstaff = "";
      for (_i = 0, _len = staff.length; _i < _len; _i++) {
        user = staff[_i];
        if (user.permission > 1) {
          staffAfk = now.getTime() - data.users[user.id].getLastActivity().getTime();
          if (staffAfk < (60 * 60 * 1000)) {
            stringstaff += "@" + user.username + " ";
          }
        }
      }
      if (stringstaff.length === 0) {
        stringstaff = "Não ha ADMs ativos no momento :'(";
      }
      return stringstaff;
    };

    staffCommand.prototype.functionality = function() {
      var currentTime, millisecondsPassed, thestaff;

      thestaff = this.staff();
      currentTime = new Date();
      if (!window.lastActiveStaffTime) {
        API.sendChat(thestaff);
        return window.lastActiveStaffTime = currentTime;
      } else {
        millisecondsPassed = currentTime.getTime() - window.lastActiveStaffTime.getTime();
        if (millisecondsPassed > 10000) {
          window.lastActiveStaffTime = currentTime;
          return API.sendChat(thestaff);
        }
      }
    };

    return staffCommand;
	
  })(Command);

  lockskipCommand = (function(_super) {
    __extends(lockskipCommand, _super);

    function lockskipCommand() {
      _ref34 = lockskipCommand.__super__.constructor.apply(this, arguments);
      return _ref34;
    }

    lockskipCommand.prototype.init = function() {
      this.command = '!lockskip';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'bouncer';
    };

    lockskipCommand.prototype.functionality = function() {
      return data.lockBooth(function() {
        return setTimeout(function() {}, API.moderateForceSkip(), setTimeout(function() {
          return data.unlockBooth();
        }, 5000), 5000);
      });
    };

    return lockskipCommand;
		 
	 })(Command);

  addCommand = (function(_super) {
    __extends(addCommand, _super);

    function addCommand() {
      _ref35 = addCommand.__super__.constructor.apply(this, arguments);
      return _ref35;
    }

    addCommand.prototype.init = function() {
      this.command = '!add';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'bouncer';
    };

    addCommand.prototype.functionality = function() {
      var msg, name, r, user;

      msg = this.msgData.message;
      if (msg.length > this.command.length + 2) {
        name = msg.substr(this.command.length + 2);
        r = new RoomHelper();
        user = r.lookupUser(name);
        if (user !== false) {
          API.moderateAddDJ(user.id);
          return setTimeout((function() {
            return data.unlockBooth();
          }), 5000);
        }
      }
    };

    return addCommand;

  })(Command);

  swapCommand = (function(_super) {
    __extends(swapCommand, _super);

    function swapCommand() {
      _ref18 = swapCommand.__super__.constructor.apply(this, arguments);
      return _ref18;
    }

    swapCommand.prototype.init = function() {
      this.command = '!swap';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'bouncer';
    };

    swapCommand.prototype.functionality = function() {
      var msg, r, swapRegex, userAdd, userRemove, users;
      msg = this.msgData.message;
      swapRegex = new RegExp("^!swap @(.+) para @(.+)$");
      users = swapRegex.exec(msg).slice(1);
      r = new RoomHelper();
      if (users.length === 2) {
        userRemove = r.lookupUser(users[0]);
        userAdd = r.lookupUser(users[1]);
        if (userRemove === false || userAdd === false) {
          API.sendChat('Ha algum erro');
          return false;
        } else {
          return data.lockBooth(function() {
            API.moderateRemoveDJ(userRemove.id);
            API.sendChat("Removendo " + userRemove.username + "...");
            return setTimeout(function() {
              API.moderateAddDJ(userAdd.id);
              API.sendChat("Adicionando " + userAdd.username + "...");
              return setTimeout(function() {
                return data.unlockBooth();
              }, 1500);
            }, 1500);
          });
        }
      } else {
        return API.sendChat("Tire o espaço entre os nomes e coloque \'para\'");
      }
    };

    return swapCommand;

  })(Command);

  popCommand = (function(_super) {
    __extends(popCommand, _super);

    function popCommand() {
      _ref19 = popCommand.__super__.constructor.apply(this, arguments);
      return _ref19;
    }

    popCommand.prototype.init = function() {
      this.command = '!pop';
      this.parseType = 'exact';
      return this.rankPrivelege = 'mod';
    };

    popCommand.prototype.functionality = function() {
      var djs, popDj;
      djs = API.getDJs();
      popDj = djs[djs.length - 1];
      return API.moderateRemoveDJ(popDj.id);
    };

    return popCommand;

  })(Command);

  pushCommand = (function(_super) {
    __extends(pushCommand, _super);

    function pushCommand() {
      _ref20 = pushCommand.__super__.constructor.apply(this, arguments);
      return _ref20;
    }

    pushCommand.prototype.init = function() {
      this.command = '!push';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'mod';
    };

    pushCommand.prototype.functionality = function() {
      var msg, name, r, user;
      msg = this.msgData.message;
      if (msg.length > this.command.length + 2) {
        name = msg.substr(this.command.length + 2);
        r = new RoomHelper();
        user = r.lookupUser(name);
        if (user !== false) {
          return API.moderateAddDJ(user.id);
        }
      }
    };

    return pushCommand;

  })(Command);

  resetAfkCommand = (function(_super) {
    __extends(resetAfkCommand, _super);

    function resetAfkCommand() {
      _ref21 = resetAfkCommand.__super__.constructor.apply(this, arguments);
      return _ref21;
    }

    resetAfkCommand.prototype.init = function() {
      this.command = '!resetafk';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'mod';
    };

    resetAfkCommand.prototype.functionality = function() {
      var id, name, u, _ref22;
      if (this.msgData.message.length > 10) {
        name = this.msgData.message.substring(11);
        _ref22 = data.users;
        for (id in _ref22) {
          u = _ref22[id];
          if (u.getUser().username === name) {
            u.updateActivity();
            API.sendChat('@' + u.getUser().username + ' seu tempo AFK foi reiniciado.');
            return;
          }
        }
        API.sendChat('' + name + ' Não pode ser encontrado');
      } else {
        API.sendChat('Mas, wat?');
      }
    };

    return resetAfkCommand;

  })(Command);

  forceSkipCommand = (function(_super) {
    __extends(forceSkipCommand, _super);

    function forceSkipCommand() {
      _ref22 = forceSkipCommand.__super__.constructor.apply(this, arguments);
      return _ref22;
    }

    forceSkipCommand.prototype.init = function() {
      this.command = '!forceskip';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'cohost';
    };

    forceSkipCommand.prototype.functionality = function() {
      var msg, param;
      msg = this.msgData.message;
      if (msg.length > 11) {
        param = msg.substr(11);
        if (param === 'Ativar') {
          data.forceSkip = true;
          return API.sendChat("Pulo forçado ativado.");
        } else if (param === 'Desativar') {
          data.forceSkip = false;
          return API.sendChat("Pulo forçado desativado.");
        }
      }
    };

    return forceSkipCommand;

  })(Command);

  overplayedCommand = (function(_super) {
    __extends(overplayedCommand, _super);

    function overplayedCommand() {
      _ref23 = overplayedCommand.__super__.constructor.apply(this, arguments);
      return _ref23;
    }

    overplayedCommand.prototype.init = function() {
      this.command = '!overplayed';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    overplayedCommand.prototype.functionality = function() {
      return API.sendCha("Acho que não, em?");
    };

    return overplayedCommand;

  })(Command);

  uservoiceCommand = (function(_super) {
    __extends(uservoiceCommand, _super);

    function uservoiceCommand() {
      _ref24 = uservoiceCommand.__super__.constructor.apply(this, arguments);
      return _ref24;
    }

    uservoiceCommand.prototype.init = function() {
      this.command = ['/uservoice()', '/idea()'];
      this.parseType = 'exact';
      return this.rankPrivelege = 'cohost';
    };

    uservoiceCommand.prototype.functionality = function() {
      var msg;
      msg = 'Have an idea for the room, our bot, or an event?  Awesome! Submit it to our uservoice and we\'ll get started on it: http://is.gd/IzP4bA';
      msg += ' (please don\'t ask for mod)';
      return API.sendChat(msg);
    };

    return uservoiceCommand;

  })(Command);

  skipCommand = (function(_super) {
    __extends(skipCommand, _super);

    function skipCommand() {
      _ref25 = skipCommand.__super__.constructor.apply(this, arguments);
      return _ref25;
    }

    skipCommand.prototype.init = function() {
      this.command = '!pula';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    skipCommand.prototype.functionality = function() {
      return API.moderateForceSkip();
    };

    return skipCommand;

  })(Command);

  whyMehCommand = (function(_super) {
    __extends(whyMehCommand, _super);

    function whyMehCommand() {
      _ref26 = whyMehCommand.__super__.constructor.apply(this, arguments);
      return _ref26;
    }

    whyMehCommand.prototype.init = function() {
      this.command = '!cotas';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    whyMehCommand.prototype.functionality = function() {
      var msg;
      msg = "/em acaba de ativar o modo cotas! ";
      msg += "Roubou sua vaga na faculdade, suas mulheres";
      return API.sendChat(msg);
    };

    return whyMehCommand;

  })(Command);

  commandsCommand = (function(_super) {
    __extends(commandsCommand, _super);

    function commandsCommand() {
      _ref27 = commandsCommand.__super__.constructor.apply(this, arguments);
      return _ref27;
    }

    commandsCommand.prototype.init = function() {
      this.command = '!commands';
      this.parseType = 'exact';
      return this.rankPrivelege = 'bouncer';
    };

    commandsCommand.prototype.functionality = function() {
      var allowedUserLevels, c, cc, cmd, msg, user, _i, _j, _len, _len1, _ref28, _ref29;
      allowedUserLevels = [];
      user = API.getUser(this.msgData.fromID);
      window.capturedUser = user;
      if (user.permission > 5) {
        allowedUserLevels = ['user', 'mod', 'host'];
      } else if (user.permission > 2) {
        allowedUserLevels = ['user', 'mod'];
      } else {
        allowedUserLevels = ['user'];
      }
      msg = '';
      for (_i = 0, _len = cmds.length; _i < _len; _i++) {
        cmd = cmds[_i];
        c = new cmd('');
        if (_ref28 = c.rankPrivelege, __indexOf.call(allowedUserLevels, _ref28) >= 0) {
          if (typeof c.command === "string") {
            msg += c.command + ', ';
          } else if (typeof c.command === "object") {
            _ref29 = c.command;
            for (_j = 0, _len1 = _ref29.length; _j < _len1; _j++) {
              cc = _ref29[_j];
              msg += cc + ', ';
            }
          }
        }
      }
      msg = msg.substring(0, msg.length - 2);
      return API.sendChat(msg);
    };

    return commandsCommand;

  })(Command);

  disconnectLookupCommand = (function(_super) {
    __extends(disconnectLookupCommand, _super);

    function disconnectLookupCommand() {
      _ref28 = disconnectLookupCommand.__super__.constructor.apply(this, arguments);
      return _ref28;
    }

    disconnectLookupCommand.prototype.init = function() {
      this.command = '!dcmembros';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'bouncer';
    };

    disconnectLookupCommand.prototype.functionality = function() {
      var cmd, dcHour, dcLookupId, dcMeridian, dcMins, dcSongsAgo, dcTimeStr, dcUser, disconnectInstances, givenName, id, recentDisconnect, resp, u, _i, _len, _ref29, _ref30;
      cmd = this.msgData.message;
      if (cmd.length > 11) {
        givenName = cmd.slice(11);
        _ref29 = data.users;
        for (id in _ref29) {
          u = _ref29[id];
          if (u.getUser().username === givenName) {
            dcLookupId = id;
            disconnectInstances = [];
            _ref30 = data.userDisconnectLog;
            for (_i = 0, _len = _ref30.length; _i < _len; _i++) {
              dcUser = _ref30[_i];
              if (dcUser.id === dcLookupId) {
                disconnectInstances.push(dcUser);
              }
            }
            if (disconnectInstances.length > 0) {
              resp = u.getUser().username + ' desconectou ' + disconnectInstances.length.toString() + ' vez';
              if (disconnectInstances.length === 1) {
                resp += '. ';
              } else {
                resp += 'es. ';
              }
              recentDisconnect = disconnectInstances.pop();
              dcHour = recentDisconnect.time.getHours();
              dcMins = recentDisconnect.time.getMinutes();
              if (dcMins < 10) {
                dcMins = '0' + dcMins.toString();
              }
              dcMeridian = dcHour % 12 === dcHour ? 'AM' : 'PM';
              dcTimeStr = '' + dcHour + ':' + dcMins + ' ' + dcMeridian;
              dcSongsAgo = data.songCount - recentDisconnect.songCount;
              resp += 'Sua desconexão mais recente foi as ' + dcTimeStr + ' (' + dcSongsAgo + ' músicas atras). ';
              if (recentDisconnect.waitlistPosition !== void 0) {
                resp += 'Ele estava ' + recentDisconnect.waitlistPosition + ' músicas';
                if (recentDisconnect.waitlistPosition > 1) {
                  resp += '';
                }
                resp += ' antes da cabine de DJ.';
              } else {
                resp += 'Ele não estava na cabine de dj ou na lista de espera.';
              }
              API.sendChat(resp);
              return;
            } else {
              API.sendChat("" + u.getUser().username + " não desconectou.");
              return;
            }
          }
        }
        return API.sendChat("'Este nome " + givenName + " não pode ser encontrado'.");
      }
    };

    return disconnectLookupCommand;

  })(Command);

  voteRatioCommand = (function(_super) {
    __extends(voteRatioCommand, _super);

    function voteRatioCommand() {
      _ref29 = voteRatioCommand.__super__.constructor.apply(this, arguments);
      return _ref29;
    }

    voteRatioCommand.prototype.init = function() {
      this.command = '!voteratio';
      this.parseType = 'startsWith';
      return this.rankPrivelege = 'bouncer';
    };

    voteRatioCommand.prototype.functionality = function() {
      var msg, name, r, u, votes;
      r = new RoomHelper();
      msg = this.msgData.message;
      if (msg.length > 12) {
        name = msg.substr(12);
        u = r.lookupUser(name);
        if (u !== false) {
          votes = r.userVoteRatio(u);
          msg = u.username + " :+1: " + votes['woot'].toString() + " vezes";
          if (votes['woot'] === 1) {
            msg += ', ';
          } else {
            msg += ', ';
          }
          msg += "e :-1: " + votes['meh'].toString() + " vezes";
          if (votes['meh'] === 1) {
            msg += '. ';
          } else {
            msg += '. ';
          }
          msg += "O seu voteratio é: " + votes['positiveRatio'].toString() + ".";
          return API.sendChat(msg);
        } else {
          return API.sendChat("Este nome' " + name +  " não pode ser encontrado'");
        }
      } else {
        return API.sendChat("Faça isso direito, newbie!");
      }
    };

    return voteRatioCommand;

  })(Command);

  avgVoteRatioCommand = (function(_super) {
    __extends(avgVoteRatioCommand, _super);

    function avgVoteRatioCommand() {
      _ref30 = avgVoteRatioCommand.__super__.constructor.apply(this, arguments);
      return _ref30;
    }

    avgVoteRatioCommand.prototype.init = function() {
      this.command = '!avgvoteratio';
      this.parseType = 'exact';
      return this.rankPrivelege = 'mod';
    };

    avgVoteRatioCommand.prototype.functionality = function() {
      var averageRatio, msg, r, ratio, roomRatios, uid, user, userRatio, votes, _i, _len, _ref31;
      roomRatios = [];
      r = new RoomHelper();
      _ref31 = data.voteLog;
      for (uid in _ref31) {
        votes = _ref31[uid];
        user = data.users[uid].getUser();
        userRatio = r.userVoteRatio(user);
        roomRatios.push(userRatio['positiveRatio']);
      }
      averageRatio = 0.0;
      for (_i = 0, _len = roomRatios.length; _i < _len; _i++) {
        ratio = roomRatios[_i];
        averageRatio += ratio;
      }
      averageRatio = averageRatio / roomRatios.length;
      msg = "Contabilidade de " + roomRatios.length.toString() + " votos por usuário, e a taxa média da sala é de " + averageRatio.toFixed(2).toString() + " votos.";
      return API.sendChat(msg);
    };

    return avgVoteRatioCommand;

  })(Command);

  cmds = [cookieCommand, newSongsCommand, whyWootCommand, themeCommand, rulesCommand, roomHelpCommand, sourceCommand, wootCommand, badQualityCommand, downloadCommand, afksCommand, allAfksCommand, statusCommand, unhookCommand, dieCommand, reloadCommand, lockCommand, unlockCommand, swapCommand, popCommand, pushCommand, overplayedCommand, uservoiceCommand, whyMehCommand, skipCommand, commandsCommand, resetAfkCommand, forceSkipCommand, disconnectLookupCommand, voteRatioCommand, avgVoteRatioCommand];

  chatCommandDispatcher = function(chat) {
    var c, cmd, _i, _len, _results;
    chatUniversals(chat);
    _results = [];
    for (_i = 0, _len = cmds.length; _i < _len; _i++) {
      cmd = cmds[_i];
      c = new cmd(chat);
      if (c.evalMsg()) {
        break;
      } else {
        _results.push(void 0);
      }
    }
    return _results;
  };

  updateVotes = function(obj) {
    data.currentwoots = obj.positive;
    data.currentmehs = obj.negative;
    return data.currentcurates = obj.curates;
  };

  announceCurate = function(obj) {
    return announceCurate;
  };

  handleUserJoin = function(user) {
    data.userJoin(user);
    data.users[user.id].updateActivity();
    return handleUserJoin;
  };

  handleNewSong = function(obj) {
    var songId;
    data.intervalMessages();
    if (data.currentsong === null) {
      data.newSong();
    } else {
      data.newSong();
      document.getElementById("button-vote-positive").click();
    }
    if (data.forceSkip) {
      songId = obj.media.id;
      return setTimeout(function() {
        var cMedia;
        cMedia = API.getMedia();
        if (cMedia.id === songId) {
          return API.moderateForceSkip();
        }
      }, obj.media.duration * 1000);
    }
  };

  handleVote = function(obj) {
    data.users[obj.user.id].updateActivity();
    return data.users[obj.user.id].updateVote(obj.vote);
  };

  handleUserLeave = function(user) {
    var disconnectStats, i, u, _i, _len, _ref31;
    disconnectStats = {
      id: user.id,
      time: new Date(),
      songCount: data.songCount
    };
    i = 0;
    _ref31 = data.internalWaitlist;
    for (_i = 0, _len = _ref31.length; _i < _len; _i++) {
      u = _ref31[_i];
      if (u.id === user.id) {
        disconnectStats['waitlistPosition'] = i - 1;
        data.setInternalWaitlist();
        break;
      } else {
        i++;
      }
    }
    data.userDisconnectLog.push(disconnectStats);
    return data.users[user.id].inRoom(false);
  };

  antispam = function(chat) {
    var plugRoomLinkPatt, sender;
    plugRoomLinkPatt = /(\bhttps?:\/\/(www.)?plug\.dj[-A-Z0-9+&@#\/%?=~_|!:,.;]*[-A-Z0-9+&@#\/%=~_|])/ig;
    if (plugRoomLinkPatt.exec(chat.message)) {
      sender = API.getUser(chat.fromID);
      if (!sender.ambassador && !sender.moderator && !sender.owner && !sender.superuser) {
        if (!data.users[chat.fromID]["protected"]) {
          API.sendChat("Sem divulgar outras salas vlw flw!!!");
          return API.moderateDeleteChat(chat.chatID);
        } else {
          return API.sendChat("Eu deveria expulsa-lo, mas estamos aqui para nos divertir");
       }
      }
    }
  };

   fans = function(chat) {
    var msg;

    msg = chat.message.toLowerCase();
    if (msg.indexOf('¨¨¨¨¨¨') !== -1 || msg.indexOf('¨¨¨¨¨¨') !== -1 || msg.indexOf('¨¨¨¨¨¨') !== -1 || msg.indexOf('¨¨¨¨¨¨') !== -1 || msg.indexOf(':trollface:') !== -1 || msg.indexOf('autowoot:') !== -1) {
      return API.moderateDeleteChat(chat.chatID);
    }
  };
  
  beggar = function(chat) {
    var msg, r, responses;
    msg = chat.message.toLowerCase();
    responses = ["@{beggar}  ", "@{beggar} ", "@{beggar} ", "@{beggar} "];
    r = Math.floor(Math.random() * responses.length);
    if (msg.indexOf('¨¨¨¨') !== -1 || msg.indexOf('¨¨¨¨') !== -1 || msg.indexOf('¨¨¨¨') !== -1 || msg.indexOf('¨¨¨¨¨¨') !== -1 || msg.indexOf('¨¨¨¨¨¨') !== -1) {
      return API.sendChat(responses[r].replace("{beggar}", chat.from));
    }
  };

  chatUniversals = function(chat) {
    data.activity(chat);
    antispam(chat);
    return beggar(chat);
  };

  hook = function(apiEvent, callback) {
    return API.on(apiEvent, callback);
  };

  unhook = function(apiEvent, callback) {
    return API.off(apiEvent, callback);
  };

  apiHooks = [
    {
      'event': API.ROOM_SCORE_UPDATE,
      'callback': updateVotes
    }, {
      'event': API.CURATE_UPDATE,
      'callback': announceCurate
    }, {
      'event': API.USER_JOIN,
      'callback': handleUserJoin
    }, {
      'event': API.DJ_ADVANCE,
      'callback': handleNewSong
    }, {
      'event': API.VOTE_UPDATE,
      'callback': handleVote
    }, {
      'event': API.CHAT,
      'callback': chatCommandDispatcher
    }, {
      'event': API.USER_LEAVE,
      'callback': handleUserLeave
    }
  ];

  initHooks = function() {
    var pair, _i, _len, _results;
    _results = [];
    for (_i = 0, _len = apiHooks.length; _i < _len; _i++) {
      pair = apiHooks[_i];
      _results.push(hook(pair['event'], pair['callback']));
    }
    return _results;
  };

  undoHooks = function() {
    var pair, _i, _len, _results;
    _results = [];
    for (_i = 0, _len = apiHooks.length; _i < _len; _i++) {
      pair = apiHooks[_i];
      _results.push(unhook(pair['event'], pair['callback']));
    }
    return _results;
  };

  initialize();

}).call(this);

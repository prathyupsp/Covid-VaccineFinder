



<!DOCTYPE html>
<html>

<head>
<title>Covid-19 Vaccine Availability Finder by Prathyush</title>
    <meta charset="UTF-8">
    <meta name="description" content="Covid-19 Vaccine Availability Finder">
    <meta name="keywords" content="covid, vaccine, vaccineavailability">
    <meta name="author" content="Jijish Thomas">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.0/dist/css/bootstrap.min.css"
        integrity="sha384-B0vP5xmATw1+K9KRQjQERJvTumQW0nPEzvF6L/Z6nronJ3oUOFUFpCjEUQouq2+l" crossorigin="anonymous">
	  <link rel="stylesheet" href="style.css">
    <!-- Global site tag (gtag.js) - Google Analytics -->
    <script async src="https://www.googletagmanager.com/gtag/js?id=G-XLFH7YHQ85"></script>
    <script>
        window.dataLayer = window.dataLayer || [];
        function gtag() { dataLayer.push(arguments); }
        gtag('js', new Date());

        gtag('config', 'G-XLFH7YHQ85');
    </script>

    <style>
        /* Sticky footer styles
-------------------------------------------------- */
        html {
            position: relative;
            min-height: 100%;
        }

        body {
            margin-bottom: 60px;
            background: #D3E1E4;
        }

		
        .footer {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 60px;
            /* Set the fixed height of the footer here */
            line-height: 60px;
            /* Vertically center the text there */
            background-color: #f5f5f5;
        }
    </style>
</head>

<body>
    
    <div class="container mb-5">
        <div class="row">
            <div class="col-sm-3"></div>
            <div class="col-sm-6 mainBlock">
                <div class="block-pad">
    
                    <div class="imgBlock">
                        <img src="images/pngLogo.png" alt="Covid vaccine finder" class="img-responsive" />
                    </div>
                    <div class="headerName">
                        <span style="font-size: 30px;"><strong>Covid-19</strong></span> <br/> Vaccine Availability Finder
                    </div>
                    <div style="margin-top:30px;margin-bottom:30px;">
                        <div class="form-group">
                            <label for="states">State</label>
                            <select id="states" class="form-control form-field" onchange=onStateChange()>
                            </select>
                        </div>
                        <div class="form-group">
                            <label for="district">District</label>
                            <select id="district" class="form-control form-field" onchange=onDistrictChange()>
                            </select>
                        </div>
                        <button onclick="getAvailability()" class="btn btn-search btn-lg btn-block"
                            id="searchButton">Search</button>
                        <button onclick="resetSearch()" class="btn btn-reset btn-lg btn-block" id="resetButton"
                            style="display: none;">Reset</button>
                    </div>
    
    
                    <div id="alert" class="alert alert-primary alert-notify" role="alert" style="display: none;">
                        <div class="row alert-texter">
                            <div class="col"> Not Available!
                                <br />
                                Retrying in <span id="timer"></span>s<br />
                            </div>
                            <div class="col"><span class="float-right">Attempt Count: <span id="attemptCount"></span> </span>
                            </div>
                        </div>
                    </div>
                </div>
                <div id="data"></div>
            </div>
            <div class="col-sm-3"></div>
        </div>
    </div>


    <footer class="footer">
        <div class="container" style="text-align: center;">
            <span class="text-muted"><span style="color:red">???</span> Social responsibility Initiative by Prathyush

            </span></br>
        </div>
    </footer>

</body>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js"
    integrity="sha512-894YE6QWD5I59HgZOGReFYm4dnWc1Qt5NtvYSaNcOP+u1T9qYdvdihz0PPSiiqn/+/3e7Jo4EaG7TubfWGUrMQ=="
    crossorigin="anonymous"></script>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@4.6.0/dist/js/bootstrap.min.js"
    integrity="sha384-+YQ4JLhjyBLPDQt//I+STsc9iw4uQqACwlvpslubQzn4u2UU2UFM80nGisd026JF"
    crossorigin="anonymous"></script>
<script>

    $(function () {
        getStates();
    })

    var attemptCount = 1;
    var myInterval;

    const baseURL = 'https://cdn-api.co-vin.in/api/v2';

    function setTimer() {
        document.getElementById("attemptCount").innerHTML = attemptCount;
        var maxTicks = 30;
        var tickCount = 0;
        var tick = function () {
            if (tickCount >= maxTicks) {
                clearInterval(myInterval);
                getAvailability();

                attemptCount++;
                document.getElementById("attemptCount").innerHTML = attemptCount;
                return;
            }
            document.getElementById("timer").innerHTML = (maxTicks - tickCount);
            tickCount++;
        };

        myInterval = setInterval(tick, 1000);
    }

    function resetSearch(){
        window.location.reload();
    }

    function getAvailability() {
        document.getElementById("searchButton").disabled = true; 
        document.getElementById("resetButton").style = "display:block"

        var stateId = $("#states option:selected").val();
        var districtId = $("#district option:selected").val();
        if (stateId > 0 && districtId > 0) {
            // var selectedDate = $("#date option:selected").val();
            var todaysDate = new Date();
            const options = { dateStyle: 'short' };
            const date = todaysDate.toLocaleString('hi-IN', options);

            document.getElementById("data").innerHTML = "";

            $.ajax({
                type: 'GET',
                url: baseURL + '/appointment/sessions/public/calendarByDistrict',
                data: {
                    "district_id": districtId,
                    "date": date
                },
                success: function (resp) {
                }
            }).done(
                function (response) {

                    let centerDatas = "";
                    if (response?.centers?.length > 0) {
                        let availabilityFound = false;

                        response.centers.forEach(center => {
                            center.sessions.forEach(session => {
                                if (Math.floor(session.available_capacity) > 0) {
                                    availabilityFound = true;
                                    center.available = true;
                                }
                            });
                        });

                        response.centers.sort(function compare(centerA, centerB) {
                            const maxAvailabilityCenterA = centerA.sessions.reduce((accumulator, session) => {
                                if (Math.floor(session.available_capacity) > 0) {
                                    if (!accumulator) {
                                        accumulator = true;
                                    }
                                }
                                return accumulator;
                            }, false);
                            const maxAvailabilityCenterB = centerB.sessions.reduce((accumulator, session) => {
                                if (Math.floor(session.available_capacity) > 0) {
                                    if (!accumulator) {
                                        accumulator = true;
                                    }
                                }
                                return accumulator;
                            }, false);

                            if (maxAvailabilityCenterA && !maxAvailabilityCenterB) {
                                return -1;
                            }
                            if (!maxAvailabilityCenterA && maxAvailabilityCenterB) {
                                return 1;
                            }
                            // a must be equal to b
                            return 0;
                        });

                        if (availabilityFound) {
                            play();
                            document.getElementById("alert").style = "display:none";
                        } else {
                            setTimer();
                            document.getElementById("alert").style = "display:block";
                        }

                        response.centers.forEach(center => {
                            centerDatas += `
                            <div class="card" style="margin: 0 10px 10px 10px; ${!center.available && 'background-color: gainsboro;'}">
                                <div class="data-head"><h5 class="card-title text-secondary card-TitleHead">${center.name} 
                                    ${center.available ?
                                        '<span class="badge badge-success">Available</span>' :
                                        '<span class="badge badge-danger">Not Available</span>'} <span class="badge badge-info">${center.fee_type} Vaccine</span></h5>
                                </div>
                               <div class="p-3">
                                <p><img src="images/home.png" alt="time" height="18" width="18" class="img-responsive"/><span class="card-address"> Address</span>: ${center.address} </p>
                                <p><img src="images/clock.png" alt="time" height="18" width="18" class="img-responsive"/> ${center.from} - ${center.to}</p>
                            
                               
                                <span>${getSessions(center.sessions)}</span>
                            </div>
                                </div>
                        `
                        })
                    } else {
                        alert("No Centers are available!");

                    }
                    document.getElementById("data").innerHTML = centerDatas;
                }).fail(function (data) {
                    alert("failed");
                });
        } else {
            alert("Select state and district!");
        }
    }

    function getSessions(sessions) {
        var sessionsData = "";
        if (sessions?.length > 0) {
            sessions.forEach(session => {
                if (Math.floor(session.available_capacity) > 0) {

                    sessionsData +=
                        `
                       
                    <h5 class="card-TitleHead session-area"> Sessions </h5>
                    <div class="bord-btm"></div>
                    <h5 class="text-primary card-TitleHead avail-num">Available Capacity : ${Math.floor(session.available_capacity)}</h5>
                    
                    <span>${getSlots(session)}</span>
                   
                    `
                }
            })
        }
        return sessionsData;
    }

    function getSlots(session) {
        var slotsData = "";

        if (session?.slots?.length > 0 && Math.floor(session.available_capacity) > 0) {
            slotsData +=
                `
                    <p><span class="card-address">Date </span>: ${session.date}</p>
                    <p><span class="card-address">Min Age Limit</span> : ${session.min_age_limit}</p>
                    <p><span class="card-address">Time Slots</span> </p>
                `
            session.slots.forEach(slot => {
                slotsData +=
                    `
                   
                    <p>${slot}</p>
                    
                `
            })
        }
        return slotsData;
    }

    function onStateChange(params) {
        getDistricts();
        document.getElementById("alert").style = "display:none";
        clearInterval(myInterval);
        document.getElementById("searchButton").disabled = false;
        // document.getElementById("data").innerHTML = '';
    }

    function onDistrictChange(params) {
        document.getElementById("alert").style = "display:none";
        clearInterval(myInterval);
        document.getElementById("searchButton").disabled = false;
        document.getElementById("data").innerHTML = '';
    }

    function getDistricts() {
        document.getElementById("data").innerHTML = "";
        var stateId = $("#states option:selected").val();
        $.ajax({
            type: 'GET',
            url: baseURL + '/admin/location/districts/' + stateId,
            success: function (resp) {
            }
        }).done(
            function (response) {
                let districtsOptionsHTML = "";
                if (response?.districts?.length > 0) {
                    response.districts.forEach(element => {
                        districtsOptionsHTML += `
                    <option value="${element.district_id}"> ${element.district_name} </option>`
                    });
                    document.getElementById("district").innerHTML = districtsOptionsHTML;
                }
            }).fail(function (data) {
                alert("failed");
            });
    }

    function getStates() {
        $.ajax({
            type: 'GET',
            url: baseURL + '/admin/location/states',

            success: function (resp) {
            }
        }).done(
            function (response) {
                let statesOptionsHTML = "<option selected> Select State </option> ";
                if (response?.states?.length > 0) {
                    response.states.forEach(element => {
                        statesOptionsHTML += `
                    <option value="${element.state_id}"> ${element.state_name} </option>`
                    });
                    document.getElementById("states").innerHTML = statesOptionsHTML;
                }
            }).fail(function (data) {
                alert("failed");
            });
    }

    function play() {
        var audio = new Audio('alert.mp3');
        audio.play();
    }
</script>



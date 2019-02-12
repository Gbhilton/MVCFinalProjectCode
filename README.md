# Live Project

## Introduction
For the last two weeks of my time at the tech academy, I worked with my peers in a team developing a full scale MVC/MVVM Web Application in C#. Working on a legacy codebase was a great learning oppertunity for fixing bugs, cleaning up code, and adding requested features. There were some big changes that could have been a large time sink, but we used what we had to deliver what was needed on time. I saw how a good developer works with what they have to make a quality product. I worked on several [back end stories](#back-end-stories) that I am very proud of. Because much of the site had already been built, there were also a good deal of [front end stories](#front-end-stories) and UX improvements that needed to be completed, all of varying degrees of difficulty. Everyone on the team had a chance to work on front end and back end stories. Over the two week sprint I also had the opportunity to work on some other project management and team programming [skills](#other-skills-learned) that I'm confident I will use again and again on future projects.
  
Below are descriptions of the stories I worked on, along with code snippets and navigation links. I also have some full code files in this repo for the larger functionalities I implemented.

## Back End Stories
* [Admin Inbox Time Off](#admin-inbox-time-off)
* [Time Off Event View](#time-off-event-view)

### Admin Inbox Time Off
***Admin Inbox that displays functioning Time Off Requests with Approve/Deny ability.***

-Added a new View Model to bring in Message and TimeOffEvent Properties for the Inbox Admin View.

	public class MessageTimeOffViewModel
   	 {
       		 public IEnumerable<Message> Messages { get; set; }
        	 public IEnumerable<TimeOffEvent> TimeOffEvents { get; set; }
    	 }

- Added controller functions for the user inbox ("Inbox") and the admin inbox ("InboxAdmin")

***I changed the "Index" view for the admin, which was just an inbox view for admin only. I then added a new Admin Inbox view, and removed the Index view entirely. This allowed, when the user/admin clicked
to see their messages, it would determine if the logged in user was a normal user or an admin and route them to the correct view. See below.***

***Changed From***
public ActionResult Index()
{
     if (User.IsInRole("Admin"))
         {
             return View(db.Messages.ToList());
          }
     //Added "Shared" to this, before it was "~/Views/AdminError.cshtml" and could not find that location.
            else return View("~/Views/Shared/AdminError.cshtml");
}
***To Below***
// GET: Message/Inbox for Users
public ActionResult Inbox()
        {
            if (User.Identity.IsAuthenticated)
            {
                if (User.IsInRole("Admin"))
                {
                    return RedirectToAction("InboxAdmin");
                }
                var userid = HttpContext.User.Identity.GetUserId();
                var messagesList = db.Messages.ToList();
                var userMessages = new List<Message>();
                foreach (var message in messagesList)
                {
                    foreach (var Id in message.RecipientList)
                    {
                        if (Id == Guid.Parse(userid))
                        {
                            userMessages.Add(message);
                        }
                    }
                }
                return View(userMessages);
            }
            else return View("LoginError");
        }

***Added a completely new Admin Inbox view to dispaly more than just messages like the User Inbox. The Admin Inbox also displays Time Off Requests and the ability to approve/deny them. This code also filters the view to 
only show Time Off Requests that have an empty "Approver ID", which then limits the view to only open requests. This method used the new View Model created above to pass in properties from both Messages and TimeOffEvents Models.***

// GET: Message/Inbox for Admin
        public ActionResult InboxAdmin()
        {
            if (User.Identity.IsAuthenticated)
            {
                if (User.IsInRole("Admin"))
                {
                var userid = HttpContext.User.Identity.GetUserId();
                var messagesList = db.Messages.Where(x => x.Sender.Id == userid);
                var timeOffEvent = db.TimeOffEvents.Where(x => x.ApproverId == null);
                var viewModel = new MessageTimeOffViewModel();

                viewModel.Messages = messagesList;
                viewModel.TimeOffEvents = timeOffEvent;
                return View(viewModel);
                }
                else return View("Inbox");
            }
            else return View("LoginError");
        }

-Added the ability to approve or deny time off requests from the Admin Inbox in the Message Controller. Once approved or denied, the view is "Refreshed" and the approved/denied request no longer shows in the view.

        public ActionResult Approve(Guid id)
        {
            TimeOffEvent timeOffEvent = db.TimeOffEvents.Find(id);
            timeOffEvent.ApproverId = new Guid(User.Identity.GetUserId());
            db.SaveChanges();
            return RedirectToAction("InboxAdmin");
        }

        
        public ActionResult Deny(Guid id, Guid userId)
        {
            TimeOffEvent timeOffEvent = db.TimeOffEvents.Find(id);
            db.TimeOffEvents.Remove(timeOffEvent);
            db.SaveChanges();
            return RedirectToAction("InboxAdmin");
        }
		
-Added an Inbox Admin view that had an additional section of "Time Off Requests", the regular inbox for the user does not have this view. I also updated through HTML the layout for a better UI.

@model Schedule_It_2.Models.MessageTimeOffViewModel

<h2>Inbox</h2>
<h4>Admin</h4>

<p class="createMessageBtn">
<p class="btn btn-default">
    @Html.ActionLink("Send Message", "Create")
</p>
</p>

<br />
<h4>Messages</h4>

<table class="table">
    <tr>
        <th>
           Date Sent
        </th>
        <th>
           Date Read
        </th>
        <th>
           Message Title
        </th>
        <th></th>
    </tr>

    @foreach (var item in Model.Messages)
    {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.DateSent)
            </td>
            <td class="readDate">
                @Html.DisplayFor(modelItem => item.DateRead)
            </td>
            <td>
                <span class="message-title">@Html.DisplayFor(modelItem => item.MessageTitle)</span>
            </td>
            <td hidden class="messageContent">
                @Html.DisplayFor(modelItem => item.Content)
            </td>
        </tr>
    }
</table>

<br />
<h4>Time Off Requests</h4>

<table class="table">
    <tr>
        <th>
            User Id
        </th>
        <th>
            Title
        </th>
        <th>
            Note
        </th>
        <th>
            Start Date
        </th>
        <th>
            End Date
        </th>
        <th>
            Approval Decision
        </th>
        <th></th>
    </tr>

    @foreach (var item in Model.TimeOffEvents)
    {
    <tr>
        <td>
            @Html.DisplayFor(modelItem => item.UserId)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.Title)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.Note)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.Start)
        </td>
        <td>
            @Html.DisplayFor(modelItem => item.End)
        </td>
        <td>
            @Html.ActionLink("Approve", "Approve", new { id = item.EventId})
            @Html.ActionLink("Deny", "Deny", new { id = item.EventId})
        </td>
    </tr>
    }
</table>

@* Modal that shows the contents of the message *@
<div id="inboxModal" class="modal">
    <div class="modal-content">
        <h4 id="modalTitle">Error</h4>
        @*<p>From: <span id="modalFrom"></span></p>*@
        <p id="modalContent">There was an error getting the message information</p>
        <div class="btn btn-default closeModal">Close</div>
    </div>
</div>

### Time Off Event View
***Time Off Request View should only display open time off requests that have not been approved/denied yet.***

-Added a return view filter in the Time Off Even controller that would only show time off requests that had an empty "Approver ID", which means there was no approve/deny response yet. Once approved/denied, when the view was refreshed, that
specific time off requests would no longer be displayed in the view.

***Changed From***
if (User.IsInRole("Admin"))
        {
			ViewBag.headerData = new TimeOffEvent();
			return View(db.TimeOffEvents.OrderBy(x => x.Start).ToList());
        }
    else return View("~/Views/Shared/AdminError.cshtml");
}

***To Below***
if (User.IsInRole("Admin"))
        {
			ViewBag.headerData = new TimeOffEvent();
			return View(db.TimeOffEvents.Where(x => x.ApproverId == null));
        }
    else return View("~/Views/Shared/AdminError.cshtml");
}

*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Other Skills](#other-skills-learned), [Page Top](#live-project)*


## Front End Stories
* [Internal Message Button](#internal-message-button)
* [Remove Delete Capability](#remove-delete-capability)

### Internal Message Button

-Added a "button" to the inbox of both the user and the admin to create messages to send internally. Added a new class in CSS to the Site.CSS file to allow for customization of the new message button.

***Changed From***
-<p>
    @Html.ActionLink("Create New", "Create")
-</p>

***To Below***
-<p class="createMessageBtn">
-<p class="btn btn-default">
    @Html.ActionLink("Send Message", "Create")
-</p>
-</p>

***CSS***
-/* Styling the "New Message" button in the inbox and index views */
.createMessageBtn {
    text-decoration: none;
    margin-bottom: 20px;
}

### Remove Delete Capability
***Removed capability to delete a message once sent/created.***

-Removed the delete function from the previous version. Removed the delete/edit ability from the view, as well as removing the delete/edit view files from previous developers. 

***Commented out the below code.***
-<td>
    //Removed the option to delete or edit a message once it has been sent.
    @*Html.ActionLink("Delete", "Delete", new { id = item.MessageId })*@
-</td>

*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Other Skills](#other-skills-learned), [Page Top](#live-project)*


## Bug Fixes/Errors

-I rerouted the view to the correct file. When I started this project the error routing was not working and didn't route to the correct view, thus displaying an error webpage.

	else return View("~/Views/AdminError.cshtml");
//Added "Shared" to this, before it was "~/Views/AdminError.cshtml" and could not find that location.
	else return View("~/Views/Shared/AdminError.cshtml");

-When I started this project, the past developers were using an "Index" as an Admin Inbox. It was an inbox, but only for admin. However it was misleading and unnecessary, thus I created a user inbox and an admin inbox view. Since
at the time, the index was identical to the user inbox. I created a admin inbox because it had a completely different view and function after that. There were multiple reroute throughout the project that needed fixing. There were
also issues when inside a messages, one could click "return to inbox", and for admin it had it direct back to inbox. So if you were an admin, in index view, and you clicked back to inbox, it rerouted back to inbox and not index. Overall it
was unneeded functionality that was resolved by this and adding an admin inbox view instead.

	return RedirectToAction("Index");
//Changed the Redirect to "Inbox" instead of "Index". This would cause an error if you were not an admin. Now once you create the message it brings you back to your inbox.
	return RedirectToAction("Inbox");

*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Other Skills](#other-skills-learned), [Page Top](#live-project)*


## Other Skills Learned
* Working with a group of developers to identify front and back end bugs to the improve usability of an application.
* Improving project flow by communicating about who needs to check out which files for their current story.
* Learning new efficiencies from other developers by observing their workflow and asking questions.  
* Practice with team programming/pair programming when one developer runs into a bug they cannot solve.
 
*Jump to: [Front End Stories](#front-end-stories), [Back End Stories](#back-end-stories), [Other Skills](#other-skills-learned), [Page Top](#live-project)*

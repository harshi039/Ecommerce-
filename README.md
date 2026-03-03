<td id="service-{{$svc.Id}}">
    {{if or (ne $svc.PlanStatus "") (ne $svc.Status "") (ne $svc.TestStatus "")}}
        <small>
            {{if ne $svc.PlanStatus ""}}
                Plan: <strong>{{$svc.PlanStatus}}</strong><br/>
            {{end}}
            {{if ne $svc.Status ""}}
                Apply: <strong>{{$svc.Status}}</strong><br/>
            {{end}}
            {{if ne $svc.TestStatus ""}}
                Test: <strong>{{$svc.TestStatus}}</strong>
            {{end}}
        </small>
    {{else}}
        <small>No Status</small>
    {{end}}
</td>
